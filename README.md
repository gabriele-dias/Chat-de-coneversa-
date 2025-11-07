Chat em Tempo Real com Filtragem de Conteúdo 

Projeto: Chat em Tempo Real com Filtragem de Conteúdo (Flask)
Este projeto implementa um sistema de chat em tempo real com autenticação de usuários e um mecanismo de filtragem de palavras banidas persistente em banco de dados. O projeto é construído utilizando o micro-framework Flask em Python.
1. Propósito e Funcionalidades
﻿
O objetivo principal do projeto é fornecer uma plataforma de comunicação instantânea que garanta um ambiente de conversação minimamente moderado.
Funcionalidades Principais:
Funcionalidade
Descrição
Implementação
Autenticação de Usuário
Permite que novos usuários se registrem e que usuários existentes façam login.
Rotas /register e /login. As senhas são armazenadas com hash SHA256.
Chat em Tempo Real
Permite a troca de mensagens instantâneas entre os usuários conectados.
Utiliza WebSockets através do Flask-SocketIO para comunicação bidirecional.
Filtro de Palavras Banidas
Bloqueia o envio de mensagens que contenham termos considerados impróprios ou proibidos.
A lista de palavras banidas é consultada no banco de dados (BannedWord model) e a filtragem é aplicada no handler de mensagens do SocketIO.
Gerenciamento de Palavras
Permite adicionar novas palavras à lista de termos banidos.
Rota /add_banned_word (requisição RESTful).
2. Tecnologias e Bibliotecas Utilizadas
O projeto é estruturado em torno do ecossistema Python e utiliza bibliotecas específicas para cada camada de funcionalidade.
2.1. Tecnologias de Backend e Estrutura
Tecnologia
Uso no Projeto
Flask
Micro-framework web principal para roteamento e gerenciamento da aplicação.
Flask-SocketIO
Extensão essencial para habilitar a comunicação em tempo real via WebSockets.
eventlet
Biblioteca de concorrência que permite ao SocketIO lidar com múltiplas conexões simultâneas de forma eficiente.
Flask-SQLAlchemy
Extensão para gerenciamento do banco de dados, facilitando a definição de modelos e a interação com o banco.
SQLite
Banco de dados leve e baseado em arquivo (instance/chat.db) utilizado para persistir dados de usuários e palavras banidas.
hashlib
Módulo padrão do Python utilizado para gerar o hash SHA256 das senhas dos usuários.
2.2. Estrutura de Dados (Modelos)
O banco de dados é definido em app/models.py e possui dois modelos principais:
Modelo
Campos (Colunas)
Propósito
User
id, username, password
Armazena as credenciais dos usuários para o sistema de login.
BannedWord
id, word
Armazena a lista de palavras que devem ser bloqueadas no chat.
3. Aspectos de Rede e Comunicação
O projeto utiliza uma arquitetura de rede híbrida, combinando protocolos para garantir a funcionalidade completa.
3.1. Protocolos de Comunicação
Protocolo
Uso Principal
Função no Projeto
HTTP (Hypertext Transfer Protocol)
Comunicação tradicional de requisição-resposta.
Utilizado para o carregamento inicial das páginas, envio de formulários de Login e Registro, e requisições RESTful (como a rota /add_banned_word).
WebSocket
Comunicação persistente, bidirecional e em tempo real.
Essencial para a funcionalidade de Chat em Tempo Real. Permite que o servidor envie dados para o cliente (e vice-versa) a qualquer momento.
3.2. Comunicação em Tempo Real (WebSockets)
A biblioteca Flask-SocketIO gerencia a camada de rede do WebSocket, utilizando o conceito de eventos para a troca de mensagens.
•
Conexão: O cliente inicia a conexão WebSocket, que é aceita pelo servidor (executando o SocketIO com eventlet), estabelecendo um link persistente.
•
Eventos: O envio de mensagens de chat é tratado pelo evento message.
•
Broadcast: O servidor utiliza a função emit(..., broadcast=True) para enviar mensagens a todos os clientes conectados, garantindo a instantaneidade da comunicação.
3.3. Servidor de Aplicação e Concorrência
O projeto utiliza o eventlet como worker de concorrência. Isso é crucial para um aplicativo de chat, pois permite que o servidor gerencie milhares de conexões WebSocket simultâneas de forma eficiente, usando um modelo de programação assíncrona (não-bloqueante).
4. Fluxo de Filtragem de Mensagens
O mecanismo de filtragem é o ponto central do projeto e funciona da seguinte maneira:
1.
Recebimento da Mensagem: O servidor SocketIO recebe a mensagem do usuário.
2.
Consulta ao Banco de Dados: O sistema consulta a lista de palavras banidas (BannedWord model) no banco de dados (chat.db).
3.
Verificação de Conteúdo: A mensagem é verificada se contém qualquer uma das palavras banidas.
4.
Ação de Bloqueio: Se uma palavra banida for encontrada, a mensagem original não é enviada para os outros usuários. Uma mensagem de "Mensagem bloqueada (conteúdo impróprio)" é enviada de volta apenas para o usuário que tentou enviar.
5.
Ação de Envio: Se a mensagem for limpa, ela é transmitida para todos os usuários conectados (broadcast=True).
