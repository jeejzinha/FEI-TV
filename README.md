# FEI-TV
**Disciplina:** CCP230 - Desenvolvimento de Algoritmos  
**Professora:** Gabriela Biondi  
**Jessica Hernandez**  
**RA** - 52.225.001-8

---

## 1. Introdução
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;O FeiTV e um sistema de streaming de videos desenvolvido em linguagem C como projeto final da disciplina de Desenvolvimento de Algoritmos. O objetivo do projeto foi criar uma aplicacao de terminal funcional que simula as principais funcionalidades de plataformas de streaming modernas, como busca de conteudo, gerenciamento de playlists e sistema de curtidas, com persistencia de dados em arquivos de texto.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;O projeto foi desenvolvido individualmente e abrange conceitos fundamentais da programacao em C, incluindo manipulacao de arquivos, structs, strings, modularizacao com funcoes e gerenciamento de estado de sessao de usuario.

---

## 2. Descrição do Sistema
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;O FeiTV e executado inteiramente via terminal e oferece ao usuario uma experiencia organizada por menus. O sistema e composto por tres grandes areas funcionais: autenticacao de usuarios, navegacao de conteudo e gerenciamento de preferencias pessoais.

### 2.1 Autenticação
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;O sistema possui tela de boas-vindas com duas opcoes: criar conta ou fazer login. Os dados dos usuarios sao armazenados no arquivo usuarios.txt no formato:  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`username;senha`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;O cadastro valida se o username escolhido ja existe no arquivo antes de permitir o registro. O login compara as credenciais digitadas com as armazenadas e, em caso de sucesso, inicia uma sessao guardando o username em uma variavel global, que e utilizada por todas as funcionalidades personalizadas do sistema.

### 2.2 Catálogo de Vídeos
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Os videos disponiveis na plataforma sao lidos do arquivo videos.txt, que possui cabecalho e entradas no formato:  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`titulo;duracao;descricao;canal`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;A busca e realizada de forma case-insensitive, ou seja, o usuario pode digitar o titulo em qualquer combinacao de maiusculas e minusculas que o sistema encontra os resultados correspondentes. Ao selecionar um video, sao exibidos seus dados completos: titulo, duracao, descricao e canal.

### 2.3 Curtidas
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Cada usuario pode curtir ou descurtir videos. As curtidas sao armazenadas no arquivo curtidos.txt no formato:  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`username;titulo_do_video`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;O sistema verifica automaticamente se o usuario ja curtiu o video exibido, mostrando o status atualizado e adaptando as opcoes do submenu (curtir ou descurtir). Na tela de videos curtidos, o usuario ve apenas seus proprios videos curtidos e pode remove-los diretamente da lista.

### 2.4 Playlists
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;O sistema de playlists permite ao usuario criar colecoes personalizadas de videos. Os dados sao armazenados no arquivo playlists.txt no formato:  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`username;nome_da_playlist;titulo_do_video`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;As principais operacoes disponiveis sao:

- Criar nova playlist com nome personalizado
- Abrir uma playlist e visualizar seus videos
- Adicionar um video a uma playlist diretamente da tela de detalhes do video
- Remover um video de uma playlist, com renumeracao automatica
- Excluir uma playlist inteira

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Todas as playlists sao exclusivas de cada usuario — nenhum usuario ve ou acessa as playlists de outro.

---

## 3. Estrutura do Codigo
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;O projeto e composto por um unico arquivo main.c, organizado em secoes bem definidas por comentarios. A estrutura segue uma abordagem modular com funcoes especializadas para cada responsabilidade.

### 3.1 Variavel de Sessao
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Uma variavel global armazena o username do usuario logado:

```c
char usuario_logado[100] = "";
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Essa variavel e preenchida no momento do login bem-sucedido e limpa ao retornar ao menu inicial (logout). Ela e utilizada por todas as funcoes que dependem da identidade do usuario, como curtidas e playlists.

### 3.2 Struct Video
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Os dados de cada video sao representados pela seguinte estrutura:

```c
typedef struct {
    char titulo[100];
    char duracao[10];
    char descricao[200];
    char canal[100];
} Video;
```

### 3.3 Persistencia em Arquivos
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Toda a persistencia de dados e feita por meio de arquivos .txt. As operacoes de escrita utilizam o modo de abertura `"a"` (append) para adicionar registros, enquanto as operacoes de exclusao reescrevem o arquivo completo omitindo o registro a ser removido — tecnica padrao em C para edicao de arquivos de texto plano.

---

## 4. Arquivos de Dados
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;O sistema utiliza quatro arquivos de texto para persistencia:

- `usuarios.txt` — credenciais de acesso de todos os usuarios cadastrados
- `videos.txt` — catalogo completo de videos disponibilizados na plataforma
- `curtidos.txt` — registro de curtidas por usuario
- `playlists.txt` — videos organizados em playlists por usuario

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Os arquivos curtidos.txt e playlists.txt sao criados automaticamente pelo programa na primeira vez que o usuario realiza a acao correspondente. O arquivo videos.txt deve ser fornecido previamente com o catalogo de conteudo, e o usuarios.txt e criado automaticamente no primeiro cadastro.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Todos os arquivos devem estar na mesma pasta que o executavel feitv.exe para que o sistema os localize corretamente em tempo de execucao.

---

## 5. Dificuldades Encontradas e Solucoes Adotadas

### 5.1 Leitura de Arquivos com fscanf
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Um dos principais desafios foi a leitura correta de arquivos com fscanf. O formato `%[^\n]` nao consome o caractere de nova linha ao final da leitura, fazendo com que iteracoes subsequentes do loop travassem no `\n` residual. A solucao foi adicionar um espaco antes do especificador de formato (`" %99[^;]"`), que instrui o fscanf a ignorar automaticamente espacos em branco e quebras de linha antes de iniciar a leitura do proximo campo.

### 5.2 Busca Case-Insensitive
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;A busca por titulo precisava funcionar independentemente de maiusculas e minusculas. A solucao foi implementar a funcao `to_lower`, que cria uma copia da string em letras minusculas sem modificar o original, e entao aplica `strstr` para verificar se o termo buscado esta contido no titulo convertido.

### 5.3 Exclusao de Registros em Arquivos
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;A linguagem C nao oferece mecanismo nativo para remover linhas de um arquivo de texto. A solucao adotada foi a releitura completa do arquivo para memoria, seguida da reescrita omitindo o registro indesejado — tecnica aplicada nas funcoes `descurtir`, `remover_video_playlist` e `excluir_playlist`.

### 5.4 Referencia Circular entre Funcoes
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;A funcao `abrir_playlist` chama `playlists` (para voltar a lista) e `playlists` chama `abrir_playlist` (para abrir uma playlist selecionada). Essa referencia circular exigiu o uso de prototipos de funcao declarados no inicio do arquivo, garantindo que o compilador conheca as assinaturas antes de encontrar as chamadas.

---

## 6. Conclusao
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;O desenvolvimento do FeiTV permitiu aplicar na pratica os conceitos estudados ao longo da disciplina de Desenvolvimento de Algoritmos. O projeto evoluiu de forma incremental, com cada funcionalidade sendo implementada, testada e corrigida antes de avancar para a proxima.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;O resultado e um sistema de terminal funcional que simula as principais interacoes de uma plataforma de streaming: autenticacao segura por usuario, busca de conteudo, curtidas individuais e gerenciamento completo de playlists personalizadas, com persistencia de todos os dados entre sessoes.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;O projeto tambem foi uma oportunidade de lidar com restricoes reais de ambiente — como a compatibilidade com C89 no Dev-C++ e o comportamento de fscanf — desenvolvendo a habilidade de diagnosticar e resolver problemas de forma sistematica.

### Vídeo do Projeto Funcionando

[🎥 Clique aqui para ver o vídeo do projeto](https://youtube.com/shorts/c-EYRRN4zNE)
---
