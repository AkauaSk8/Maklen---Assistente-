<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>MAKLEN - ASSISTENTE VIRTUAL</title>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;700&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Inter', sans-serif;
            background: linear-gradient(to bottom, #004aad, #0077ff);
            margin: 0;
            height: 100vh;
            display: flex;
            justify-content: center;
            align-items: center;
            color: #333;
        }
        .chat-container {
            width: 100%;
            max-width: 450px;
            background: #fff;
            border-radius: 20px;
            box-shadow: 0 10px 30px rgba(0,0,0,0.2);
            display: flex;
            flex-direction: column;
            overflow: hidden;
            height: 95vh;
            margin: 20px;
        }
        .chat-header {
            background: #004aad;
            color: #fff;
            padding: 20px;
            text-align: center;
            font-weight: bold;
            font-size: 1.25rem;
            border-bottom: 3px solid #0059b3;
            border-radius: 20px 20px 0 0;
        }
        .chat-body {
            flex: 1;
            padding: 15px;
            overflow-y: auto;
            background: #f0f4f9;
            display: flex;
            flex-direction: column;
            gap: 10px;
        }
        .message {
            padding: 12px 18px;
            border-radius: 20px;
            max-width: 85%;
            line-height: 1.4;
            font-size: 1rem;
            animation: fadeIn 0.3s ease-in-out;
        }
        .user {
            background: #e6f3ff;
            align-self: flex-end;
            border-bottom-right-radius: 8px;
            color: #1a5276;
        }
        .bot {
            background: #f7f7f7;
            align-self: flex-start;
            border-bottom-left-radius: 8px;
            color: #555;
        }
        .typing {
            font-style: italic;
            color: #999;
            margin: 5px;
            align-self: flex-start;
        }
        .chat-footer {
            display: flex;
            padding: 15px;
            background: #eee;
            border-top: 1px solid #ddd;
        }
        .chat-footer input {
            flex: 1;
            padding: 12px 20px;
            border-radius: 25px;
            border: 1px solid #ccc;
            font-size: 1rem;
            transition: all 0.3s;
        }
        .chat-footer input:focus {
            outline: none;
            border-color: #0077ff;
            box-shadow: 0 0 5px rgba(0,119,255,0.3);
        }
        .chat-footer button {
            margin-left: 10px;
            padding: 12px 20px;
            border: none;
            border-radius: 25px;
            background: #004aad;
            color: white;
            font-weight: bold;
            cursor: pointer;
            transition: background 0.3s;
        }
        .chat-footer button:hover {
            background: #003a8d;
        }
        .menu-buttons {
            display: flex;
            flex-direction: column;
            gap: 10px;
            margin: 10px 0;
        }
        .menu-buttons button {
            padding: 12px 18px;
            border: none;
            border-radius: 15px;
            background: #0077ff;
            color: #fff;
            font-weight: bold;
            cursor: pointer;
            transition: background 0.2s, transform 0.2s;
            box-shadow: 0 4px 8px rgba(0,0,0,0.1);
        }
        .menu-buttons button:hover {
            background: #0056cc;
            transform: translateY(-2px);
        }
        @keyframes fadeIn {
            from { opacity: 0; transform: translateY(10px); }
            to { opacity: 1; transform: translateY(0); }
        }
    </style>
</head>
<body>
    <div class="chat-container">
        <div class="chat-header">🤖 MAKLEN - ASSISTENTE VIRTUAL</div>
        <div class="chat-body" id="chatBody"></div>
        <div class="chat-footer">
            <input type="text" id="userInput" placeholder="Digite sua resposta..." onkeydown="if(event.key==='Enter'){sendMessage()}">
            <button onclick="sendMessage()">Enviar</button>
        </div>
    </div>

    <script>
        let userName = null;
        let esperandoNome = true;
        let menuAtivo = false;
        let modoSimulacao = false;
        let esperandoSenha = false;
        let chatBody = document.getElementById("chatBody");
        let userInput = document.getElementById("userInput");

        const SENHA = "MK123***";

        const dicas = [
            "1️⃣ Ouça o cliente com atenção e não o interrompa.",
            "2️⃣ Demonstre empatia e compreensão pela situação.",
            "3️⃣ Ofereça soluções viáveis e que se encaixem na realidade do cliente.",
            "4️⃣ Mantenha a calma, mesmo em situações difíceis, e seja profissional.",
            "5️⃣ Mostre os benefícios de resolver a pendência, como a melhoria do crédito."
        ];

        const respostas = {
            "desempregado": ["Entendo, e lamento por essa fase. Podemos montar algo simbólico e parcelado.", "Sinto muito por sua situação. Podemos buscar uma proposta especial para esse momento."],
            "doente": ["Sinto muito, sei que não é fácil. Posso ajustar a menor condição possível.", "Lamento muito por sua saúde. Priorize-a. Assim que puder, voltamos a conversar."],
            "não posso pagar": ["Eu entendo. Mas se definirmos algo hoje, você garante condições melhores.", "Entendo sua situação, mas a melhor forma de resolver é encontrando um valor que caiba no seu orçamento agora."],
            "sem dinheiro": ["Temos parcelamentos leves que cabem em qualquer orçamento.", "Compreendo, mas temos opções com parcelas bem pequenas para te ajudar."],
            "não vou pagar": ["Respeito sua posição, mas posso verificar se há um desconto maior para você.", "Entendo. Mas a dívida pode crescer com juros. Vamos tentar encontrar uma solução agora para evitar problemas futuros."],
            "não é minha dívida": ["Entendo. Podemos confirmar os dados para garantir que não haja engano.", "Para garantir que a cobrança é devida, podemos conferir seus dados. Qual seu CPF?"],
            "vou processar": ["Compreendo sua frustração. O objetivo do meu contato é evitar qualquer tipo de transtorno, buscando um acordo justo.", "Não precisa chegar a esse ponto. Estou aqui para te ajudar a resolver a situação de forma amigável."],
            "o banco não me ajuda": ["Estou aqui para ser a ponte entre você e o banco. Me diga qual a sua melhor proposta para que eu possa levar a eles.", "Sou seu ponto de contato para resolver isso. Juntos, vamos encontrar uma proposta viável."],
            "juros abusivos": ["Entendo sua preocupação com os juros. Por isso, temos condições especiais que podem reduzir significativamente esse valor.", "Você tem razão. Por isso, as propostas de hoje já vêm com um grande desconto nos juros."],
            "não reconheço": ["Nesse caso, podemos abrir um protocolo de contestação. Para isso, preciso confirmar alguns dados com você.", "Certo, é importante investigar. O que você não reconhece na cobrança?"],
            "já negociei": ["Ótimo! Pode me informar o número do acordo para que eu possa verificar?", "Se já negociou, vamos localizar o acordo no sistema. Por favor, me passe a data ou o número da negociação."],
            "me ligam toda hora": ["Peço desculpas pelo incômodo. Posso agendar um horário mais conveniente para nosso próximo contato?", "Entendo o transtorno. Para que isso não aconteça mais, vamos resolver essa pendência agora. O que acha?"],
            "preciso pensar": ["Claro, é importante tomar essa decisão com calma. Mas saiba que a oferta de hoje pode não ser a mesma amanhã. Qual seria a sua maior dúvida?", "Entendo. Para te ajudar a pensar, posso te enviar a proposta por e-mail, e podemos agendar para falarmos novamente em um ou dois dias."],
            "não tenho proposta": ["Entendo, mas para te ajudar a encontrar a melhor solução, preciso de um valor inicial. Quanto você poderia pagar hoje ou no próximo mês?", "Para eu poder te ajudar, preciso de uma estimativa. Com quanto você poderia iniciar a negociação?"],
            "dívida é antiga": ["O fato da dívida ser antiga significa que temos mais flexibilidade para negociar. Essa é a oportunidade perfeita para você quitar com um grande desconto.", "Justamente por ser antiga, ela já teve juros. Por isso, as propostas de hoje são as melhores para você se livrar dela de vez."],
            "não tenho tempo": ["Entendo que sua rotina é corrida. Podemos agendar essa ligação para um horário que seja melhor para você. Qual o melhor dia e hora?", "Para não tomar seu tempo, vamos ser rápidos. Qual o valor que você consegue pagar para a gente já fechar o acordo?"],
            "quitação": ["Claro, podemos negociar a quitação. Qual valor você tem disponível para pagar a dívida à vista?", "Para quitação à vista, conseguimos um desconto maior. Qual valor você consegue para a gente liquidar a dívida?"],
            "parcelar": ["Podemos analisar a melhor opção de parcelamento para você. Qual valor de parcela caberia no seu bolso por mês?", "Temos planos de parcelamento flexíveis. Para te ajudar, preciso saber o valor que você consegue pagar por mês."],
            "vou pensar": ["Certo, é uma decisão importante. Para te ajudar, posso enviar a proposta por e-mail ou WhatsApp para que você possa analisar com calma. Qual seu e-mail ou WhatsApp?", "O que te impede de fechar hoje? Se me disser, posso tentar te ajudar a resolver."],
            "valor alto": ["Eu entendo, o valor total da dívida pode assustar. Mas justamente por isso, estamos oferecendo um grande desconto para que você possa pagar um valor que seja justo.", "O valor pode ser alto, mas as parcelas são pequenas. Qual valor de parcela você consegue pagar?"],
            "já paguei": ["Ok, se você já pagou, pode me enviar o comprovante para que eu possa dar baixa no sistema? Se não tiver, posso te ajudar a localizar o pagamento.", "Se a dívida já foi paga, preciso do comprovante para dar baixa. Pode me enviar para que eu possa resolver?"],
            "nome sujo": ["Sei que ter o nome negativado é difícil. A boa notícia é que com esse acordo você pode quitar sua dívida e limpar seu nome em poucos dias!", "Esse acordo é a sua chance de limpar seu nome e voltar a ter crédito no mercado. Vamos fechar?"],
            "não recebi boleto": ["Peço desculpas pelo transtorno. Posso te enviar a segunda via do boleto agora mesmo. Qual é a melhor forma de envio: e-mail ou WhatsApp?", "Sem problemas, posso gerar um novo boleto e te enviar agora. Qual é o melhor canal para você receber?"],
            "negociação anterior": ["Para te ajudar, posso verificar as condições da sua última negociação e ver se consigo uma proposta ainda melhor para você.", "Vamos consultar a última negociação para ver se consigo liberar uma condição mais vantajosa para você."],
            "não confio": ["Compreendo, mas pode confiar. Estou ligando da empresa XYZ, com o objetivo de te ajudar a resolver essa situação de forma segura e transparente.", "Entendo sua desconfiança. Estou te ligando do setor de cobrança, e podemos confirmar os dados para sua segurança."],
            "piorou minha vida": ["Sinto muito que a situação tenha chegado a esse ponto. A melhor forma de reverter essa situação é negociando. Estou aqui para te ajudar.", "Vamos resolver isso juntos. Minha função é te ajudar a sair dessa situação e não deixar que a dívida atrapalhe ainda mais sua vida."],
            "não tenho garantia": ["A sua única garantia sou eu e a seriedade da nossa empresa. Se você aceitar a proposta, vou te enviar um documento oficial com todas as condições do acordo.", "A garantia é a documentação. Assim que fecharmos o acordo, vou te enviar o documento com todos os detalhes. Pode confiar."],
            "crise": ["Entendo, estamos em um momento de crise. E é por isso que estamos aqui para ajudar. Não queremos que essa dívida se torne uma bola de neve. Vamos encontrar uma solução juntos.", "Em tempos de crise, a organização financeira é fundamental. Vamos negociar para que você não tenha mais essa preocupação."],
            "fui enganado": ["Compreendo sua frustração. Podemos analisar o ocorrido e encontrar a melhor solução para você. Meu objetivo é resolver a pendência da melhor forma possível.", "Lamento o que aconteceu. Podemos abrir um processo de investigação, mas para isso precisamos resolver a dívida primeiro."],
            "estou trabalhando": ["Entendo que está ocupado(a). Qual o melhor horário para eu te ligar de volta? Posso te ligar em 1 ou 2 horas?", "Sei que o tempo é corrido. Posso te ligar mais tarde? Por favor, me diga o melhor horário."],
            "ligação caiu": ["Estou ligando de volta pois a nossa ligação caiu. Podemos continuar? Onde estávamos na conversa?", "Olá, a ligação caiu. Vamos retomar. Qual foi o último valor que você conseguiu visualizar?"],
            "não aguento mais": ["Eu entendo sua exaustão. Mas saiba que este é o último passo para você se livrar dessa preocupação. Vamos fechar o acordo agora?", "É compreensível que esteja cansado. A boa notícia é que podemos resolver isso em poucos minutos e você não terá mais essa dor de cabeça."],
            "me ligue depois": ["Claro, posso te ligar depois. Qual o melhor horário e dia? Posso te ligar no final da tarde ou amanhã pela manhã?", "Sem problemas. Qual o melhor horário para você? Posso te ligar em uma ou duas horas?"],
            "não tenho proposta": ["Para te ajudar, preciso de um valor inicial. Quanto você poderia pagar hoje?", "Me diga o valor que você pode pagar por mês, e eu busco a melhor proposta para você."],
            "procon": ["Compreendo sua frustração. Não precisa chegar a esse ponto. O objetivo é resolver o problema de forma amigável.", "Nossa meta é resolver isso sem burocracia. Vamos fechar um acordo para que você não precise acionar o Procon."],
            "ameaça": ["Entendo que se sinta ameaçado(a), mas nossa intenção é apenas te ajudar a resolver essa pendência. Acredite, estamos aqui para somar e não para atrapalhar.", "Sinto muito se a abordagem foi inadequada. Não é nossa intenção ameaçar. Vamos resolver a situação de forma amigável."],
            "salário atrasado": ["Entendo que essa situação com seu salário seja complicada. Mas podemos negociar com base no valor que você tem disponível hoje, não no valor total da dívida.", "Sei que é uma situação difícil. Podemos fechar um acordo que se encaixe no seu orçamento, mesmo com o salário atrasado."],
            "não me lembro": ["Entendo. Para te ajudar a lembrar, podemos repassar os detalhes da dívida. Você gostaria de verificar a data de compra ou os valores?", "Sem problemas. Posso te passar as informações da dívida para que você possa conferir no seu sistema. Qual o seu CPF?"],
            "telefone errado": ["Me desculpe, por favor. Pode me informar o número correto para que eu possa atualizar em nosso sistema?", "Agradeço por me informar. Vou atualizar nosso cadastro para que você não receba mais ligações."],
            "fui bloqueado": ["Compreendo sua frustração. Vamos resolver a situação agora para que seu acesso seja desbloqueado.", "É compreensível. Mas a melhor forma de ter o acesso liberado é regularizando a pendência."],
            "morte na família": ["Sinto muito pela sua perda. O luto é uma fase muito difícil, e respeitamos isso. Posso te ligar em alguns dias?", "Meus pêsames. Sinto muito. Vamos conversar mais tarde, em um momento mais oportuno para você."],
            "não sabia": ["Entendo. Por isso estou ligando, para te informar sobre a pendência e te ajudar a resolver. As informações estão corretas?", "Não sabia? Agradeço por me informar. Vou verificar e te ligar mais tarde."],
            "gasto não autorizado": ["Compreendo. Vamos abrir um protocolo de investigação. Mas para que o valor não acumule, vamos encontrar uma forma de negociar a dívida para que a pendência não cresça ainda mais."],
            "já é judicial": ["Entendo. A pendência já está em fase judicial. Mas, se houver interesse, ainda podemos negociar e fechar um acordo.", "Sei que a dívida está em fase judicial. Mas podemos tentar encontrar uma proposta para resolver de forma amigável."],
            "idoso": ["Entendo. Para garantir a segurança dos seus dados, posso conversar com um responsável? Ou prefere que eu ligue mais tarde?", "Compreendo. Você se sentiria mais seguro se eu pudesse conversar com seu filho ou responsável? Qual o melhor horário para eu ligar?"],
            "adolescente": ["Seus pais estão em casa? Posso falar com eles?", "Entendo que você é menor de idade. Vou tentar falar com o responsável pela dívida."],
            "piorou minha vida": ["Sinto muito que a situação tenha chegado a esse ponto. A melhor forma de reverter essa situação é negociando. Estou aqui para te ajudar."],
            "não tenho garantia": ["A sua única garantia sou eu e a seriedade da nossa empresa. Se você aceitar a proposta, vou te enviar um documento oficial com todas as condições do acordo."],
            "crise": ["Entendo, estamos em um momento de crise. E é por isso que estamos aqui para ajudar. Não queremos que essa dívida se torne uma bola de neve. Vamos encontrar uma solução juntos."],
            "fui enganado": ["Compreendo sua frustração. Podemos analisar o ocorrido e encontrar a melhor solução para você. Meu objetivo é resolver a pendência da melhor forma possível."],
        };

        function sendMessage(text = null) {
            const message = text || userInput.value.trim();
            if (message === "") return;

            addMessage(message, "user");
            userInput.value = "";

            if (message.toLowerCase() === "sair" || message.toLowerCase() === "voltar ao menu" || message.toLowerCase() === "voltar" || message.toLowerCase() === "iniciar") {
                resetState();
                return;
            }

            if (esperandoNome) {
                userName = message;
                esperandoNome = false;
                setTimeout(() => {
                    addMessage(`Seja bem-vindo(a), ${userName}! 👋`, "bot");
                    mostrarMenu();
                }, 800);
                return;
            }
            
            // Nova lógica para a opção oculta
            if (message.toLowerCase() === "palavras-chaves" && !esperandoSenha) {
                iniciarAcessoBloqueado();
                return;
            }

            if (esperandoSenha) {
                if (message === SENHA) {
                    addMessage("✅ Senha correta. Acesso liberado para as palavras-chave.", "bot");
                    mostrarPalavrasChave();
                } else {
                    addMessage("❌ Senha incorreta. Tente novamente ou digite 'sair' para voltar ao menu.", "bot");
                }
                esperandoSenha = false;
                return;
            }

            if (menuAtivo) {
                const choice = message.toLowerCase();
                if (choice.includes("1") || choice.includes("treinamento")) {
                    iniciarTreinamento();
                } else if (choice.includes("2") || choice.includes("simulação")) {
                    iniciarSimulacao();
                } else if (choice.includes("4") || choice.includes("voltar ao inicio")) {
                    resetState();
                } else {
                    addMessage("Opção inválida. Por favor, clique em um botão ou digite o número da opção desejada.", "bot");
                }
                return;
            }

            if (modoSimulacao) {
                processarSimulacao(message);
            }
        }

        function resetState() {
            userName = null;
            esperandoNome = true;
            menuAtivo = false;
            modoSimulacao = false;
            esperandoSenha = false;
            chatBody.innerHTML = "";
            addMessage("Olá! Sou a Maklen, sua assistente virtual. 🤖", "bot");
            setTimeout(() => {
                addMessage("Antes de começarmos, como você se chama?", "bot");
            }, 800);
        }

        function iniciarTreinamento() {
            menuAtivo = false;
            addMessage("📘 Treinamento selecionado! Aqui estão algumas dicas para você:", "bot");
            dicas.forEach((dica, index) => {
                setTimeout(() => addMessage(dica, "bot"), 500 * (index + 1));
            });
            setTimeout(() => mostrarMenu(), 500 * (dicas.length + 1));
        }

        function iniciarSimulacao() {
            modoSimulacao = true;
            menuAtivo = false;
            addMessage("🎭 Simulação iniciada! Agora eu sou o cliente. Digite sua resposta e eu vou reagir. Para voltar ao menu principal, digite 'sair' a qualquer momento.", "bot");
            setTimeout(() => {
                addMessage("Olá, estou ligando sobre uma pendência. Mas não tenho como pagar agora. O que podemos fazer?", "bot");
            }, 1000);
        }

        function iniciarAcessoBloqueado() {
            menuAtivo = false;
            esperandoSenha = true;
            addMessage("🔒 Para acessar as palavras-chave, digite a senha e pressione Enter. Para voltar ao menu, digite 'sair'.", "bot");
        }

        function mostrarPalavrasChave() {
            addMessage("--- Lista de Palavras-Chave e Respostas ---", "bot");
            let sortedKeys = Object.keys(respostas).sort();
            sortedKeys.forEach(chave => {
                addMessage(`**"${chave}"** -> ${respostas[chave].join(" | ")}`, "bot");
            });
            addMessage("--- Fim da lista. Para voltar ao menu, digite 'sair'. ---", "bot");
        }

        function processarSimulacao(text) {
            showTyping();
            const lowerText = text.toLowerCase();
            let respostaEncontrada = null;
            let melhorCorrespondencia = 0;

            for (const chave in respostas) {
                if (lowerText.includes(chave)) {
                    respostaEncontrada = respostas[chave];
                    break;
                }
            }

            if (respostaEncontrada) {
                const opcoes = respostaEncontrada;
                const respostaAleatoria = opcoes[Math.floor(Math.random() * opcoes.length)];
                setTimeout(() => {
                    removeTyping();
                    addMessage(respostaAleatoria, "bot");
                }, 1000);
            } else {
                setTimeout(() => {
                    removeTyping();
                    addMessage("Entendi. Poderia me explicar melhor sua situação para que eu possa te ajudar?", "bot");
                }, 1000);
            }
        }

        function addMessage(text, sender) {
            const message = document.createElement("div");
            message.classList.add("message", sender);
            message.innerHTML = text;
            chatBody.appendChild(message);
            chatBody.scrollTop = chatBody.scrollHeight;
        }

        function showTyping() {
            let typing = document.getElementById("typing");
            if (!typing) {
                typing = document.createElement("div");
                typing.id = "typing";
                typing.classList.add("typing");
                typing.textContent = "Digitando...";
                chatBody.appendChild(typing);
                chatBody.scrollTop = chatBody.scrollHeight;
            }
        }

        function removeTyping() {
            const typing = document.getElementById("typing");
            if (typing) typing.remove();
        }

        function mostrarMenu() {
            const existingMenu = document.querySelector('.menu-buttons');
            if (existingMenu) {
                existingMenu.remove();
            }

            const menuDiv = document.createElement("div");
            menuDiv.classList.add("menu-buttons");

            const btn1 = document.createElement("button");
            btn1.textContent = "1️⃣ Treinamento";
            btn1.onclick = () => sendMessage("1");

            const btn2 = document.createElement("button");
            btn2.textContent = "2️⃣ Simulação";
            btn2.onclick = () => sendMessage("2");

            const btn4 = document.createElement("button");
            btn4.textContent = "4️⃣ Voltar ao Início";
            btn4.onclick = () => sendMessage("4");

            menuDiv.appendChild(btn1);
            menuDiv.appendChild(btn2);
            menuDiv.appendChild(btn4);
            chatBody.appendChild(menuDiv);
            chatBody.scrollTop = chatBody.scrollHeight;

            menuAtivo = true;
        }

        window.onload = () => {
            addMessage("Olá! Sou a Maklen, sua assistente virtual. 🤖", "bot");
            setTimeout(() => {
                addMessage("Antes de começarmos, como você se chama?", "bot");
            }, 800);
        };
    </script>
</body>
</html>
