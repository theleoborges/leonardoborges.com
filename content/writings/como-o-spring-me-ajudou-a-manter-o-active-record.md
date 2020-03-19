+++ 
title = "como o spring me ajudou a manter o active record"
date = "2007-10-13"
slug = "2007/10/13/como-o-spring-me-ajudou-a-manter-o-active-record"
tags =["Architecture", "Java", "Patterns", "Spring"]
+++

<p>
Nesse primeiro post vou falar um pouco sobre meu novo trabalho. Na verdade, sobre um problema que encontramos lÃ¡.<br><br>Dentre diversas coisas que estamos fazendo, como migraÃ§Ãµes por exemplo, estou ajudando na elaboraÃ§Ã£o da arquitetura que irÃ¡ abrigar as novas aplicaÃ§Ãµes desenvolvidas na empresa. <!--more-->Ã‰ uma tarefa desgastante e cada decisÃ£o Ã© crÃ­tica, indo de discussÃµes sobre qual tecnologia usar para MVC atÃ© como organizar o <em>layering</em> da aplicaÃ§Ã£o. Essa arquitetura compreende tambÃ©m o desenvolvimento de um <em>framework </em>para facilitar a vida dos novos desenvolvedores que integrarÃ£o a equipe.<br><br>Pois bem, uma das decisÃµes que tomei foi a de evitar o <a target="_blank" href="http://martinfowler.com/bliki/AnemicDomainModel.html" title="Anemic Domain Model"><em>Anemic Domain Model</em></a><em> </em>e usar o Spring<em> </em>como framework de <a target="_blank" href="http://martinfowler.com/bliki/InversionOfControl.html" title="IoC">IoC</a>.<em> </em>Os benefÃ­cios disso sÃ£o assunto para outro post. ;) E um outro padrÃ£o que ajuda nisso se chama <em><a target="_blank" href="http://martinfowler.com/eaaCatalog/activeRecord.html" title="Active Record">Active Record</a></em>. AÃ­ comeÃ§aram os problemas! Essa divisÃ£o de responsabilidades sempre agrega uma nova (nem tÃ£o nova assim) forma de pensar. Vou mostrar um exemplo para ilustrar isso.<br><br>[java]<br>public class Usuario {<br>private String login;<br>private String senha;<br>//normal getters e setters<br>}<br>[/java]<br>Nada de novo certo? Mais uma classe burra sem lÃ³gica de negÃ³cio alguma! Pois bem, seguindo o padrÃ£o Active Record, essa classe deveria "saber" como se persistir, delegando a real lÃ³gica para um <em>Repository</em>, que nada mais Ã© que uma espÃ©cie de DAO. Desta forma, a classe que acabei de mostrar ficaria parecida com isso:<br><br>[java]<br>public class Usuario {<br>private String login;<br>private String senha;<br>// getters e setters<br><br>private Repository repo;<br><br>public void setRepository(Repository repo) {<br>this.repo = repo;<br>}<br><br>public void save();<br>public void update();<br>public void delete();<br><br>}<br>[/java]<br><br>A variÃ¡vel repo seria injetada pelo Spring, evitando assim qualquer cÃ³digo de lookup na classe UsuÃ¡rio. PorÃ©m, como vocÃª espera carregar um usuÃ¡rio previamente cadastrado no banco de dados? Recorrendo a um DAO padrÃ£o no cÃ³digo cliente - um faÃ§ade por exemplo ?<br><br>AÃ­ tudo que foi feito teria sido em vÃ£o! Se a classe sabe como se persistir, deve prover meios de se carregar objetos do seu repositÃ³rio da mesma forma, isolando totalmente detalhes de implementaÃ§Ã£o do cÃ³digo cliente. E aÃ­ comeÃ§am os problemas.<br><br>Vamos Ã  terceira versÃ£o de nossa classe:<br><br>[java]<br>public class Usuario {<br>private String login;<br>private String senha;<br>// getters e setters<br><br>private Repository repo;<br><br>public void setRepository(Repository repo) {<br>this.repo = repo;<br>}<br><br>public void save();<br>public void update();<br>public void delete();<br><br>public static Collection<usuario></usuario> findByLogin(String login) {<br>return this.repo.findByLogin(login);<br>}<br>[/java]<br><br>Ora ora ora! Espera aÃ­! Um mÃ©todo estÃ¡tico acessando uma variÃ¡vel de instÃ¢ncia?!? NÃ£o pode! E isso me tirou o sono! Como Ã© que eu vou injetar uma dependÃªncia num objeto se eu preciso dela antes mesmo de ter uma instÃ¢ncia do mesmo?<br><br>Uma maneira, porca, de se resolver isso seria alterando a variÃ¡vel repo para static e inicializando-a num bloco static, assim:<br><br>[java]<br>static {<br>repo = new Usuario().getRepo();<br>}<br>[/java]<br><br>Dois problemas com isso: Eu exponho meu dao para o mundo - tÃ¡, posso resolver alterando o modificador de acesso do mÃ©todo - e instancio um novo objeto Ãºnica e exclusivamente para pegar o DAO que o Spring injetou e joga-lo fora.<br><br>Soa estranho nÃ£o? E Ã©! NÃ£o queria ter que fazer isso. Funciona, mas Ã© feio e tosco! E agora vem mais um motivo da minha lista de coisas que eu amo no Spring! A classe <em>MethodInvokingFactoryBean</em>!<br><br>SÃ£o poucas linhas de configuraÃ§Ã£o no arquivo do contexto da aplicaÃ§Ã£o:<br><br>[xml]<br><bean id="userConfigurer" class="org.springframework.beans.factory.config.MethodInvokingFactoryBean"></bean><br><br><property value="Usuario.setRepository" name="staticMethod"></property><br><property ref="userRepo" name="arguments"></property><!-- userRepo Ã© apenas um bean comum configurado em algum ponto do arquivo, que representa o repositÃ³rio de usuÃ¡rios--><br><!-- bean-->[/xml]Dessa forma, na inicializaÃ§Ã£o do container de InversÃ£o de controle, a classe usuÃ¡rio jÃ¡ teria sua variÃ¡vel estÃ¡tica inicializada e, como nÃ£o Ã© proibido usar uma variÃ¡vel estÃ¡tica a partir de um mÃ©todo de instÃ¢ncia, os mÃ©todos save(), update() e delete() jÃ¡ estariam com suas vidas resolvidas tambÃ©m, usando o mesmo repositÃ³rio que foi inicializado no carregamento do container!Acho que fica uma soluÃ§Ã£o limpa, elegante e nÃ£o te faz abrir mÃ£o dos benefÃ­cios da inversÃ£o de controle e do padrÃ£o Active Record.Desculpem o primeiro post! Ficou meio grande nÃ©? Mas espero que tenha ficado legal! Um grande abraÃ§o e atÃ© a prÃ³xima!
</p>
