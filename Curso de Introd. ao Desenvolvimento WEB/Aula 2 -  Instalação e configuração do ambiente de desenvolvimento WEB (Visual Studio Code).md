Nesta aula vamos explorar as ferramentas necessárias para ingressar de desenvolvimento web. Ter um ambiente de desenvolvimento é essencial para garantir uma melhor qualidade e uma boa experiência no desenvolvimento de aplicações.

Durante esta aula, iremos abordar passo a passo a instalação do Visual Studio Code, um dos editores de texto mais populares e poderosos disponíveis atualmente. Assim como o Google Chrome o navegador que utilizaremos para acessar o nosso site

# Como é uma página HTML?
Antes de instalarmos as nossas ferramentas, vamos avançar um pouco e observar um exemplo da estrutura básica de uma página web, escrita em HTML, para que fique claro onde queremos chegar. 

No Ubuntu o editor de textos que vem instalado padrão é o **gedit**. Abra o programa e digite o seguinte código:

```
<!DOCTYPE html>
<html>
<head>
<title>Meu primeiro Site</title>
</head>
<body>
<div>
<h1>Oi, esse é meu primeiro site</h1>
</div>
</body>
</html>
```

Em seguida clique no botão **Salvar**, escolha uma pasta qualquer e um nome para o seu arquivo terminando com uma extensão `.html`, por exemplo: selecione a pasta `Documentos` e dê o nome para o arquivo `meu_site.html`.

O nome do arquivo não deve ter letras maiúsculas, espaços ou caracteres especiais (como `~!@#$%^&*()+[]/\|;:.?'`).

Para separar palavras, utilize o hífen (`-`) ou o subtraço (conhecido como *underline* ou *underscore*, representado por `_`). Com essas regras em mente, é possível salvar com o nome que preferir.

> [!NOTE]
> Usar caracteres especiais no nome dos arquivos cria uma confusão para os navegadores, pois alguns destes são utilizados como comandos para controlar uma página web.
> 
> Para evitar erros o navegador faz uma codificação, ou uma transformação desses caracteres. Ao salvar como **Documento sem título 1.html** o navegador entenderá como **Documento%20sem%20título%201.html**

Em seguida abra o arquivo que você acabou de salvar clicando duas vezes sobre ele e você verá o resultado no seu navegador
![[1-meu-site.png]]
E parabéns, com isso temos um exemplo funcional de um site (ou página web) funcionando no seu computador. Em seguida vamos conhecer as ferramentas necessárias para trabalhar melhor com este site.

# Google Chrome

O Google Chrome é um [[Aula 1 - História e evolução da computação e da internet#Origem da WEB e Navegadores|navegador]] de internet desenvolvido pela Google, lançado em 2008, para o Windows, e mais tarde foi portado para Linux, Mac, iOS e Android. Normalmente, cada sistema operacional já possui um navegador pré-instalado por padrão, o Windows possui o Microsoft Edge, o Linux o Mozilla Firefox, e o Safari para Mac. Porém adotaremos o Google Chrome pois este possui os mais avançados recursos da web e é o navegador mais popular da atualidade.

## Instalação
Caso o seu computador já não possua o Google Chrome instalado, você deve baixá-lo através do site: https://www.google.com/chrome e clicar em "Faça o download aqui."
![[2-google-chrome-page.png]]
No Linux, uma tela aparecerá para que você selecione o formato do arquivo adequado para a sua distribuição, no caso do Ubuntu, selecione a opção **64 bits .deb (para Debian/Ubuntu)** e em seguida clique em **Aceitar e instalar**.
![[3-google-chrome-distro.png]]
Um arquivo começará a ser baixado para o seu computador e após finalizar, abra este arquivo e em seguida clique em **Instalar**
![[4-google-chrome-install.png]]Com isso o navegador estará instalado e pronto para usar.

# Visual Studio Code
O Visual Studio Code (abreviado como VSCode) é um editor de texto/código-fonte desenvolvido pela Microsoft para Windows, Linux e macOS. Ele inclui suporte para depuração, controle de versionamento Git incorporado, realce de sintaxe, complementação inteligente de código. Exploraremos muitas das funcionalidades deste editor ao longo do curso.

## Instalação
A instalação é muito parecida com o [[Aula 2 -  Instalação e configuração do ambiente de desenvolvimento WEB (Visual Studio Code)#Google Chrome|Google Chrome]], basta acessar https://code.visualstudio.com/ e novamente escolher a versão compatível a distribuição, neste caso clicando em **.deb**.

![[5-vscode-page.png]]
Repita o processo de abrir o arquivo após baixado e em seguida clique em **Instalar**. Após alguns instantes seu VSCode estará instalado.

## Configuração
Ao abrir o VSCode você sera apresentado à algumas configurações básicas que podem ser personalizadas de acordo com a sua preferência e a explicação de algumas funcionalidades. Na tela de boas-vindas escolha o tema de cores que mais lhe agrada.
![[6-vscode-welcome.png]]
Em seguida vamos primeiramente mudar o idioma do nosso VSCode para Português do Brasil, para que o idioma não seja uma barreira durante o aprendizado. Observe um menu vertical à esquerda com vários ícones, passando o cursor do mouse por cima de cada um, você terá mais informações sobre cada um:
- Explorer (`Ctrl+Shift+E`) - Explorador.
- Search (`Ctrl+Shift+F`) - Busca.
- Source Control (`Ctrl+Shift+G`) - Controle de versão.
- Run and Debug (`Ctrl+Shift+D`) - Executar e Debugar.
- Extensions (`Ctrl+Shift+X`) - Extensões
- Accounts - Conta
- Manage - Gerenciar

Clique em **Extensions** ou pressione as teclas `Ctrl+Shift+X` juntas para abrir o menu de extensões, você verá a loja de extensões para o VSCode com várias sugestões para as mais variadas tecnologias no qual ele oferece suporte. Na barra de busca **Search Extensions in Marketplace** digite a palavra **Brazil** (em inglês, com "z").
![[8-vscode-extensions-store.png]]

Em seguida, no primeiro resultado **Portuguese (Brazil) Language Pack for Visual Studio Code** clique no botão à direita **Install**. em instantes um pop-up (tipo de aviso que surge na tela) aparecerá no canto inferior direito da tela solicitando alterar para o idioma que recém instalamos e reiniciar o VSCode, confirme clicando em **Change Language and Restart**. O editor irá reiniciar em português do brasil.

> [!NOTE]
> Você ainda poderá encontrar partes do VSCode em inglês, que ainda não foram traduzidas

## Primeiros passos

Agora, podemos tentar visualizar o [[Aula 2 -  Instalação e configuração do ambiente de desenvolvimento WEB (Visual Studio Code)#Como é uma página HTML?|arquivo que criamos anteriormente]] para ver a diferença. Para isso, no menu superior do VSCode, clique em **Arquivo** e em seguida em **Abrir o arquivo**, ou simplesmente utilize o atalho `Ctrl+O` e procure pelo arquivo HTML que criamos, que por convenção colocamos em `Documentos/meu_site.html`.

Um aviso de permissão poderá ser exibido, se este aparecer, marque a opção **Lembrar minha decisão em todos os espaços de trabalho** e em seguida clique em **Abrir**.
![[9-vscode-permission-modal.png]]

Logo que o arquivo é aberto, uma coisa deverá chamar sua atenção, algumas partes do nosso código HTML agora estão coloridos, desta forma:
```html
<!DOCTYPE html>
<html>
<head>
<title>Meu primeiro Site</title>
</head>
<body>
<div>
<h1>Oi, esse é meu primeiro site</h1>
</div>
</body>
</html>
```
Essa é uma das funcionalidades do VSCode em relação aos editores de texto comum, conhecido como *syntax highlighting* ou destaque de sintaxe, ele automaticamente entende um arquivo pela sua extensão, neste caso `.html` 

Algumas das muitas outras podem ser conferidas no próprio site do VSCode (https://code.visualstudio.com/docs/languages/html, em inglês), mas podemos destacar:

### *IntelliSense*
À medida que você digita HTML, aparecem sugestões via a funcionalidade *IntelliSense*. Na imagem abaixo, você pode ver uma sugestão de fechamento de elemento HTML `</div>` bem como uma lista específica de contexto de elementos sugeridos

![[10-vscode-intellisense.png]]

### Tags de fechamento
Os elementos da tag são fechados automaticamente quando `>` da tag de abertura é digitada.
![[11-vscode-autoclose-tags.png]]

### *Hover*
Mova o mouse sobre tags HTML ou estilos incorporados e JavaScript para obter mais informações sobre o símbolo sob o cursor.
![[12-vscode-hover.png]]