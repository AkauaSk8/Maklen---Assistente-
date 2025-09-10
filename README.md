<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <title>MAKLEN - ASSISTENTE VIRTUAL</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background: linear-gradient(to right, #004aad, #0077ff);
      margin: 0;
      height: 100vh;
      display: flex;
      justify-content: center;
      align-items: center;
    }
    .chat-container {
      width: 100%;
      max-width: 450px;
      background: #fff;
      border-radius: 15px;
      box-shadow: 0px 0px 15px rgba(0,0,0,0.3);
      display: flex;
      flex-direction: column;
      overflow: hidden;
      height: 90vh;
    }
    .chat-header {
      background: #004aad;
      color: #fff;
      padding: 15px;
      text-align: center;
      font-weight: bold;
      font-size: 18px;
    }
    .chat-body {
      flex: 1;
      padding: 15px;
      overflow-y: auto;
      background: #f4f4f9;
      display: flex;
      flex-direction: column;
    }
    .message {
      margin: 8px 0;
      padding: 10px 14px;
      border-radius: 20px;
      max-width: 80%;
      line-height: 1.4;
      font-size: 15px;
    }
    .user {
      background: #d1e7dd;
      align-self: flex-end;
      border-bottom-right-radius: 5px;
    }
    .bot {
      background: #e2e3e5;
      align-self: flex-start;
      border-bottom-left-radius: 5px;
    }
    .typing {
      font-style: italic;
      color: gray;
      margin: 5px;
    }
    .chat-footer {
      display: flex;
      padding: 10px;
      background: #eee;
    }
    .chat-footer input {
      flex: 1;
      padding: 10px;
      border-radius: 20px;
      border: 1px solid #ccc;
    }
    .chat-footer button {
      margin-left: 10px;
      padding: 10px 15px;
      border: none;
      border-radius: 20px;
      background: #004aad;
      color: white;
      font-weight: bold;
      cursor: pointer;
    }
  </style>
</head>
<body>
  <div class="chat-container">
    <div class="chat-header">🤖 MAKLEN - ASSISTENTE VIRTUAL</div>
    <div class="chat-body" id="chatBody"></div>
    <div class="chat-footer">
      <input type="text" id="userInput" placeholder="Digite a frase do cliente...">
      <button onclick="sendMessage()">Enviar</button>
    </div>
  </div>

  <script>
    const respostas = {
      "desempregado": [
        "Entendo sua situação, perder o emprego é difícil. Podemos ajustar um parcelamento leve para não deixar a dívida crescer.",
        "Mesmo desempregado, podemos começar com um valor simbólico para manter seu crédito ativo.",
        "Regularizando agora, você evita bloqueios e já deixa sua situação mais tranquila para quando conseguir um novo emprego."
      ],
      "nao posso pagar": [
        "Compreendo, mas temos opções de parcelamento que cabem no seu bolso. Qual valor seria confortável para você hoje?",
        "Podemos iniciar com uma parcela menor para evitar juros maiores. Que valor consegue destinar?",
        "Se não regularizar agora, a dívida pode crescer. Sugiro começarmos com um acordo acessível hoje."
      ],
      "quero cancelar": [
        "O cancelamento definitivo só ocorre após a quitação. Mas posso ajudá-lo a regularizar e encerrar sua dívida agora mesmo.",
        "Entendo sua vontade, mas para cancelar é necessário quitar. Posso montar uma proposta que finalize sua dívida hoje.",
        "Cancelar sem pagar gera problemas futuros. O melhor é regularizar e encerrar de vez sua obrigação."
      ],
      "nao devo": [
        "Entendo sua colocação, mas consta em nosso sistema essa pendência. Posso enviar os detalhes para sua análise.",
        "Podemos revisar juntos. Enquanto isso, é importante regularizar para evitar restrições maiores.",
        "Mesmo que haja dúvidas, recomendo garantir o acordo e depois solicitar contestação junto ao banco."
      ],
      "sem dinheiro": [
        "Sei que está apertado, mas temos parcelamentos bem leves que cabem em qualquer orçamento.",
        "Qual valor mínimo o senhor(a) consegue separar hoje para iniciarmos a regularização?",
        "Podemos adaptar a parcela ao seu bolso. O importante é dar o primeiro passo hoje."
      ],
      "agressivo": [
        "Entendo sua insatisfação, estou aqui para ajudar. Vamos buscar juntos a melhor solução.",
        "Mantenho meu respeito, e quero apenas lhe apresentar uma forma de resolver essa pendência sem maiores transtornos.",
        "Peço desculpas se ficou insatisfeito, mas meu objetivo é encontrar a melhor proposta para sua situação."
      ],
      "enrolado": [
        "Percebo que está postergando, mas quanto mais tempo passa, maiores ficam os encargos.",
        "Posso ajudar a encerrar essa dívida hoje mesmo com uma condição especial.",
        "Se resolvermos agora, você já elimina essa preocupação de uma vez."
      ]
    };

    function sendMessage() {
      const input = document.getElementById("userInput");
      const text = input.value.trim();
      if (text === "") return;

      addMessage(text, "user");
      input.value = "";

      showTyping();

      const lowerText = text.toLowerCase();
      let respostaEncontrada = null;

      for (let chave in respostas) {
        if (lowerText.includes(chave)) {
          const opcoes = respostas[chave];
          respostaEncontrada = opcoes[Math.floor(Math.random() * opcoes.length)];
          break;
        }
      }

      if (!respostaEncontrada) {
        respostaEncontrada = "Entendi, mas preciso que explique melhor sua situação para eu sugerir a melhor solução.";
      }

      setTimeout(() => {
        removeTyping();
        addMessage(respostaEncontrada, "bot");
      }, 1200);
    }

    function addMessage(text, sender) {
      const chatBody = document.getElementById("chatBody");
      const message = document.createElement("div");
      message.classList.add("message", sender);
      message.textContent = text;
      chatBody.appendChild(message);
      chatBody.scrollTop = chatBody.scrollHeight;
    }

    function showTyping() {
      const chatBody = document.getElementById("chatBody");
      const typing = document.createElement("div");
      typing.id = "typing";
      typing.classList.add("typing");
      typing.textContent = "MAKLEN está digitando...";
      chatBody.appendChild(typing);
      chatBody.scrollTop = chatBody.scrollHeight;
    }

    function removeTyping() {
      const typing = document.getElementById("typing");
      if (typing) typing.remove();
    }
  </script>
</body>
</html>
