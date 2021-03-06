+++ 
title = "refactoring for readability"
date = "2009-09-18"
slug = "2009/09/18/refactoring-for-readability"
tags =["JRuby", "Refactoring", "Ruby"]
+++

<p>
Yesterday I've done something I should do more often: Revisit some code written a while ago for our current project and make it better.<br><br>Let's face it. We all write crappy code the 1st time. The difference is in what we do about it afterwards.<br>We might decide it's good enough and keep moving, or we could (and should!) stop and refactor it!<br><br>The code I revisited worked as a refactoring exercise and it's initial version is shown below:

``` ruby Jphoto
class Jphoto
 ...

 #a few other methods ...

 def post_photo(file_data, hotel_id, send_rss, options = {})
   file_name = "tmp/#{Time.now.to_i}_#{rand(1000000).to_s(36)}"
   File.open(file_name, "wb") do |f|
     f.puts(file_data)
   end

   params = [Curl::PostField.file('photo',file_name),
      Curl::PostField.content('hotel', hotel_id),
      Curl::PostField.content('source','PhotoUploadTest')]
   extract_extra_params!(params, options)

   c = Curl::Easy.new("#{service_uri_base}/photoupld")
   c.multipart_form_post = true
   c.http_post(*params)

   if c.response_code != 200
     error_msg = "File upload failed with code: #{c.response_code}"
     Rails.logger.info error_msg
     raise error_msg
   end

   File.delete(file_name)

   hotel = Hotel.find_by_id(hotel_id)
   hotel.cache.destroy_all

   send_upload_rss(hotel, original_upload_url(c.body_str) , options) if send_rss
 end

 private

 def send_upload_rss(hotel, photo_url, options)
   ...
 end
 def manage_images_link(hotel_id)
   ...
 end

 def extract_extra_params!(params, options)
   params << Curl::PostField.content('status', options[:status]) if options[:status]
   params << Curl::PostField.content('upload_source', options[:upload_source]) if options[:upload_source]
   params << Curl::PostField.content('uploader_ip', options[:uploader_ip]) if options[:uploader_ip]
   params << Curl::PostField.content('uploader_email', options[:uploader_email]) if options[:uploader_email]
 end
end
```

<br>Look at the post_photo method. It has problems in so many levels that it's hard to start. <br>Methods should do "one thing" and that method obviously does much more than that, mixing different levels of abstraction.<br><br>But let's start with the easy parts first, keeping in mind that I was aiming for readability.<br><br>Lines 7 to 10 seem to be there just to make the reader's life harder. It's creating a temporary file through some custom logic instead of using the tools provided by the language. Unnecessary and only pollutes our eyes. My first measure was to use ruby's TempFile class for this. Better, but we still have a long way.<br><br>Right at line 12 it creates some sort of default parameters list, after which it extracts some extra options. I don't know what that method does but it's clearly using output arguments, which we should avoid at all costs, as they lead to confusion. This is a big smell as well, and another refactoring step added to my list.<br><br>On line 21 starts the code that handles what to do when we get a response_code other than 200 from our request. Apart from the fact that this code doesn't feel right here, we just happen to know that in HTTP, 200 means success, but that might not be clear to someone looking at the code for the 1st time.<br><br>Then the code goes on to delete the temp file, clear the hotel's cache and send the rss if the rss' flag is true.<br><br><strong>Let there be refactoring....</strong><br><br>Geez, how many lines have I used to explain what the code does? Since I don't wanna bore you to death, here is my refactored version of this method, trying to avoid as much as I can the problems I highlighted previously:<br><br><br><br>

``` ruby
class Jphoto
  ...
  SUCCESS = 200
  
  #a few other methods ...
  
  def post_photo(file_data, hotel_id, send_rss, options = {})
    response_body = make_post_request(file_data, hotel_id, options)
    hotel = Hotel.find_by_id(hotel_id)
    hotel.cache.destroy_all
    send_upload_rss(hotel, original_upload_url(response_body) , options) if send_rss
  end
  ...
```

Ha! That reads much nicer, right? 
Below you'll find the rest of the class, properly refactored. Note how I also changed the order of the private methods so the class has a reading flow. 
You can now read it top down, without scrolling through the file several times trying to find where the methods are defined.

``` ruby
  private
  def make_post_request(file_data, hotel_id, options)
    temp_file = Tempfile.open('minisite_upload_')
    temp_file.puts(file_data)
    c = Curl::Easy.new("#{service_uri_base']}/photoupld")
    c.multipart_form_post = true
    c.http_post(*build_params_list(temp_file.path, hotel_id, options))
    temp_file.delete
    raise_and_log_error("File upload failed with code: #{c.response_code}") if c.response_code != SUCCESS
    c.body_str
  end
  def build_params_list(file_path, hotel_id, options)
    params = [Curl::PostField.file('photo', file_path), Curl::PostField.content('hotel', hotel_id), Curl::PostField.content('source','PhotoUploadTest')]
    params << Curl::PostField.content('status', options[:status]) if options[:status]
    params << Curl::PostField.content('upload_source', options[:upload_source]) if options[:upload_source]
    params << Curl::PostField.content('uploader_ip', options[:uploader_ip]) if options[:uploader_ip]
    params << Curl::PostField.content('uploader_email', options[:uploader_email]) if options[:uploader_email]
    params
  end
  def raise_and_log_error(error_msg)
    Rails.logger.info error_msg
    raise error_msg
  end
  def original_upload_url(jphotos_response)
    ...
  end
  def send_upload_rss(hotel, photo_url, options)
    ...
  end
  def manage_images_link(hotel_id)
    ...
  end
end

```


I've only applied 2 basic principles to this code: That methods should do one thing and that they should avoid output arguments. But the result was a drastic improvement over the old code.
<strong>A last step...</strong>
There is still one thing I would like to change in the post_photo method. It takes a boolean as an argument. The first time you see a call to this method, there is no way you can tell what that flag is used for:

``` ruby
  jphoto.post_photo(file_data, 1, false, options)
```

To make it more clear we could refactor the method once more to look like this:


``` ruby
  def post_photo_and_send_rss(file_data, hotel_id, options = {})
    response_body = post_photo(file_data, hotel_id, options)
    send_upload_rss(hotel, original_upload_url(response_body) , options)
  end
  def post_photo(file_data, hotel_id, options = {})
    response_body = make_post_request(file_data, hotel_id, options)
    hotel = Hotel.find_by_id(hotel_id)
    hotel.cache.destroy_all
    response_body
  end
```	

This way we avoid an extra parameter and save our reader - and possibly us - of the trouble to guess what that flag was for.<br><br><br>I am by no means saying this is the definitive refactored state of this code but instead sharing the steps I went through while refactoring it. As always, tips and comments are welcome! :) 
</p>

