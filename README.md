# meu-app

<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>TransVid ❤️ - Vídeos pra quem precisa</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: Arial, Helvetica, sans-serif;
            background: #0a0f1e;
            padding: 20px;
            color: white;
        }

        .container {
            max-width: 700px;
            margin: auto;
        }

        .card {
            background: #1a1f2e;
            border-radius: 20px;
            padding: 20px;
            margin-bottom: 20px;
            border: 1px solid #2a2f3e;
        }

        .plan-card {
            background: linear-gradient(135deg, #1e1b4b, #0f172a);
            border-radius: 20px;
            padding: 20px;
            margin-bottom: 20px;
            text-align: center;
            border-left: 5px solid #fbbf24;
        }

        .quota-number {
            font-size: 48px;
            font-weight: bold;
            color: #fbbf24;
        }

        textarea {
            width: 100%;
            background: #0f172a;
            border: 1px solid #334155;
            border-radius: 12px;
            padding: 12px;
            color: white;
            font-size: 16px;
            font-family: inherit;
            resize: vertical;
        }

        button {
            width: 100%;
            background: #f97316;
            border: none;
            padding: 14px;
            border-radius: 40px;
            color: white;
            font-weight: bold;
            font-size: 16px;
            cursor: pointer;
            margin-top: 12px;
            transition: 0.2s;
        }

        button:hover {
            background: #ea580c;
        }

        button.secondary {
            background: #334155;
        }

        button.secondary:hover {
            background: #475569;
        }

        .flex {
            display: flex;
            gap: 10px;
            margin-top: 12px;
        }

        .flex button {
            margin-top: 0;
        }

        .hidden {
            display: none;
        }

        .scene-item {
            background: #0f172a;
            border-radius: 12px;
            padding: 12px;
            margin-bottom: 10px;
            border-left: 4px solid #fbbf24;
        }

        .success {
            background: #10b98120;
            border: 1px solid #10b981;
            border-radius: 12px;
            padding: 10px;
            margin-top: 10px;
            text-align: center;
            color: #34d399;
        }

        .watermark-box {
            background: #fbbf2420;
            border-radius: 12px;
            padding: 10px;
            margin-top: 10px;
            font-size: 14px;
            text-align: center;
        }
    </style>
</head>
<body>
<div class="container">
    <!-- Cabeçalho -->
    <div style="text-align: center; margin-bottom: 20px;">
        <h1 style="background: linear-gradient(135deg, #fbbf24, #ef4444); -webkit-background-clip: text; background-clip: text; color: transparent;">❤️ TransVid</h1>
        <p style="color: #9ca3af;">Rico de coração • Pra quem mais precisa</p>
    </div>

    <!-- Plano -->
    <div class="plan-card">
        <div id="planBadge" style="font-size: 14px;">🟡 Plano gratuito</div>
        <div class="quota-number" id="quotaCount">2</div>
        <div id="quotaText">vídeos restantes hoje</div>
        <div id="watermarkMsg" class="watermark-box">✨ Compartilhe 1 vídeo e remova a marca d'água ✨</div>
        <button id="lowIncomeBtn" style="background: #6366f1; margin-top: 12px;">❤️ Sou baixa renda (ativar 10 vídeos/dia)</button>
    </div>

    <!-- Passo 1: Ideia -->
    <div class="card">
        <h3>📝 1. Sua ideia</h3>
        <textarea id="ideaInput" rows="3" placeholder="Ex: Como fazer um currício, vender brigadeiro, passar no ENEM..."></textarea>
        <button id="generateBtn">✨ Gerar roteiro</button>
    </div>

    <!-- Passo 2: Roteiro -->
    <div class="card hidden" id="scriptCard">
        <h3>📋 2. Roteiro gerado</h3>
        <div id="scriptText" style="background: #0f172a; border-radius: 12px; padding: 15px; margin: 10px 0; line-height: 1.5;"></div>
        <button id="createScenesBtn" class="secondary">🎬 Criar cenas</button>
    </div>

    <!-- Passo 3: Cenas -->
    <div class="card hidden" id="scenesCard">
        <h3>🎨 3. Cenas + narração</h3>
        <div id="scenesList"></div>
        <button id="renderBtn">🎥 Renderizar vídeo</button>
    </div>

    <!-- Passo 4: Vídeo pronto -->
    <div class="card hidden" id="videoCard">
        <h3>✅ 4. Vídeo pronto!</h3>
        <div id="videoInfo" style="background: #0f172a; border-radius: 12px; padding: 15px; margin: 10px 0; text-align: center;"></div>
        <div class="flex">
            <button id="downloadBtn">⬇️ Baixar MP4</button>
            <button id="shareBtn">📤 Compartilhar</button>
        </div>
    </div>
</div>

<script>
    // ========================================
    // ESTADO DO APP (TUDO FUNCIONAL)
    // ========================================
    let userType = "normal"; // normal ou lowIncome
    let videosUsed = 0;
    let lastDate = localStorage.getItem("lastDate") || "";
    let watermarkRemoved = localStorage.getItem("watermarkRemoved") === "true";
    
    let currentIdea = "";
    let currentScript = "";
    let scenes = [];
    let videoRendered = false;

    // Elementos
    const planBadge = document.getElementById("planBadge");
    const quotaCount = document.getElementById("quotaCount");
    const quotaText = document.getElementById("quotaText");
    const watermarkMsg = document.getElementById("watermarkMsg");
    const lowIncomeBtn = document.getElementById("lowIncomeBtn");
    const ideaInput = document.getElementById("ideaInput");
    const generateBtn = document.getElementById("generateBtn");
    const scriptCard = document.getElementById("scriptCard");
    const scriptText = document.getElementById("scriptText");
    const createScenesBtn = document.getElementById("createScenesBtn");
    const scenesCard = document.getElementById("scenesCard");
    const scenesList = document.getElementById("scenesList");
    const renderBtn = document.getElementById("renderBtn");
    const videoCard = document.getElementById("videoCard");
    const videoInfo = document.getElementById("videoInfo");
    const downloadBtn = document.getElementById("downloadBtn");
    const shareBtn = document.getElementById("shareBtn");

    // ========== FUNÇÃO DE RESET DIÁRIO ==========
    function resetDaily() {
        const today = new Date().toISOString().split("T")[0];
        if (lastDate !== today) {
            videosUsed = 0;
            watermarkRemoved = false;
            localStorage.setItem("lastDate", today);
            localStorage.setItem("videosUsed", "0");
            localStorage.setItem("watermarkRemoved", "false");
            lastDate = today;
        } else {
            videosUsed = parseInt(localStorage.getItem("videosUsed")) || 0;
            watermarkRemoved = localStorage.getItem("watermarkRemoved") === "true";
        }
    }

    // ========== ATUALIZAR INTERFACE ==========
    function updateUI() {
        resetDaily();
        
        const maxVideos = userType === "lowIncome" ? 10 : 2;
        const remaining = Math.max(0, maxVideos - videosUsed);
        
        quotaCount.innerText = remaining;
        
        if (userType === "lowIncome") {
            planBadge.innerHTML = "🟢 MODO SOLIDÁRIO ATIVO • 10 VÍDEOS/DIA";
            quotaText.innerText = "vídeos restantes hoje (baixa renda)";
        } else {
            planBadge.innerHTML = "🟡 PLANO GRATUITO • 2 VÍDEOS/DIA";
            quotaText.innerText = "vídeos restantes hoje";
        }
        
        if (watermarkRemoved) {
            watermarkMsg.innerHTML = "✅ MARCA D'ÁGUA REMOVIDA! Você compartilhou e ajudou alguém.";
            watermarkMsg.style.background = "#10b98120";
        } else {
            watermarkMsg.innerHTML = "✨ Compartilhe 1 vídeo e remova a marca d'água pra sempre ✨";
            watermarkMsg.style.background = "#fbbf2420";
        }
    }

    // ========== VERIFICAR SE PODE CRIAR VÍDEO ==========
    function canCreate() {
        resetDaily();
        const maxVideos = userType === "lowIncome" ? 10 : 2;
        if (videosUsed >= maxVideos) {
            alert(`Limite atingido! ${maxVideos} vídeos por dia. Volte amanhã.`);
            return false;
        }
        return true;
    }

    // ========== CONTAR VÍDEO USADO ==========
    function countVideo() {
        videosUsed++;
        localStorage.setItem("videosUsed", videosUsed);
        updateUI();
    }

    // ========== MOSTRAR MENSAGEM ==========
    function showMessage(cardElement, msg, isSuccess = true) {
        const oldMsg = cardElement.querySelector(".msg-temp");
        if (oldMsg) oldMsg.remove();
        
        const msgDiv = document.createElement("div");
        msgDiv.className = "msg-temp";
        msgDiv.style.background = isSuccess ? "#10b98120" : "#ef444420";
        msgDiv.style.border = isSuccess ? "1px solid #10b981" : "1px solid #ef4444";
        msgDiv.style.borderRadius = "12px";
        msgDiv.style.padding = "10px";
        msgDiv.style.marginTop = "10px";
        msgDiv.style.textAlign = "center";
        msgDiv.style.color = isSuccess ? "#34d399" : "#f87171";
        msgDiv.innerText = msg;
        
        cardElement.appendChild(msgDiv);
        setTimeout(() => msgDiv.remove(), 3000);
    }

    // ========== GERAR ROTEIRO (IA SIMPLES) ==========
    function generateScript(idea) {
        return `📌 TÍTULO: ${idea.toUpperCase()}

🎬 ABERTURA (0:00 - 0:15)
"Fala, pessoal! Hoje eu vou te ensinar ${idea} de um jeito simples e que funciona."

📚 DESENVOLVIMENTO (0:15 - 1:30)
• Passo 1: entenda o básico
• Passo 2: pratique todo dia
• Passo 3: não desista, os resultados vêm

🎯 FINAL (1:30 - 2:00)
"Compartilhe esse vídeo com alguém que também precisa aprender ${idea}. Juntos somos mais fortes!"`;
    }

    // ========== CRIAR CENAS ==========
    function createScenes(idea) {
        return [
            { texto: "🎬 CENA 1 - Abertura", narracao: `🎙️ "Vamos começar! Hoje é dia de aprender ${idea}."` },
            { texto: "📖 CENA 2 - Conteúdo principal", narracao: `🎙️ "O segredo é consistência. Faça um pouco todo dia."` },
            { texto: "💛 CENA 3 - Mensagem final", narracao: `🎙️ "Você é capaz! Compartilhe conhecimento e ajude quem precisa."` }
        ];
    }

    // ========== EVENTOS ==========
    
    // Botão baixa renda
    lowIncomeBtn.addEventListener("click", () => {
        if (userType === "normal") {
            userType = "lowIncome";
            lowIncomeBtn.innerText = "✅ Modo solidário ativo (10 vídeos/dia)";
            lowIncomeBtn.style.background = "#10b981";
            alert("❤️ Bem-vindo ao modo solidário! Agora você tem 10 vídeos por dia. Compartilhe conhecimento!");
        } else {
            userType = "normal";
            lowIncomeBtn.innerText = "❤️ Sou baixa renda (ativar 10 vídeos/dia)";
            lowIncomeBtn.style.background = "#6366f1";
            alert("Modo normal ativado: 2 vídeos por dia.");
        }
        updateUI();
    });

    // Gerar roteiro
    generateBtn.addEventListener("click", () => {
        if (!canCreate()) return;
        
        const idea = ideaInput.value.trim();
        if (idea === "") {
            alert("Escreva uma ideia primeiro!");
            return;
        }
        
        currentIdea = idea;
        currentScript = generateScript(idea);
        scriptText.innerText = currentScript;
        scriptCard.classList.remove("hidden");
        scenesCard.classList.add("hidden");
        videoCard.classList.add("hidden");
        
        showMessage(scriptCard, "✅ Roteiro gerado com sucesso!", true);
    });

    // Criar cenas
    createScenesBtn.addEventListener("click", () => {
        if (!currentIdea) {
            alert("Gere um roteiro primeiro!");
            return;
        }
        
        scenes = createScenes(currentIdea);
        scenesList.innerHTML = "";
        
        scenes.forEach((scene, index) => {
            const div = document.createElement("div");
            div.className = "scene-item";
            div.innerHTML = `
                <strong>${scene.texto}</strong><br>
                <span style="color: #9ca3af;">🔊 ${scene.narracao}</span>
            `;
            scenesList.appendChild(div);
        });
        
        scenesCard.classList.remove("hidden");
        showMessage(scenesCard, `🎨 ${scenes.length} cenas criadas com narração!`, true);
    });

    // Renderizar vídeo
    renderBtn.addEventListener("click", async () => {
        if (!canCreate()) return;
        if (scenes.length === 0) {
            alert("Crie as cenas primeiro!");
            return;
        }
        
        renderBtn.innerText = "🖥️ Renderizando...";
        renderBtn.disabled = true;
        
        // Simula renderização
        await new Promise(resolve => setTimeout(resolve, 2000));
        
        countVideo();
        videoRendered = true;
        
        const watermarkStatus = watermarkRemoved ? "✅ SEM MARCA D'ÁGUA" : "🔸 COM MARCA D'ÁGUA (compartilhe para remover)";
        
        videoInfo.innerHTML = `
            ✅ VÍDEO FINALIZADO!<br><br>
            📹 Título: ${currentIdea}<br>
            🎙️ ${scenes.length} cenas narradas<br>
            🎨 ${watermarkStatus}<br>
            ⏱️ Duração: ~90 segundos<br><br>
            ✨ Compartilhe agora e remova a marca d'água!
        `;
        
        videoCard.classList.remove("hidden");
        renderBtn.innerText = "🎥 Renderizar vídeo";
        renderBtn.disabled = false;
        
        showMessage(videoCard, "✅ Vídeo renderizado com sucesso!", true);
    });

    // Download
    downloadBtn.addEventListener("click", () => {
        if (!videoRendered) {
            alert("Renderize o vídeo primeiro!");
            return;
        }
        
        const content = `VIDEO: ${currentIdea}\n\nROTEIRO:\n${currentScript}\n\nCENAS:\n${scenes.map(s => `${s.texto}: ${s.narracao}`).join("\n")}`;
        const blob = new Blob([content], {type: "text/plain"});
        const url = URL.createObjectURL(blob);
        const a = document.createElement("a");
        a.href = url;
        a.download = `transvid_${Date.now()}.txt`;
        a.click();
        URL.revokeObjectURL(url);
        
        alert("Download iniciado! (Demo - arquivo .txt com informações do vídeo)");
    });

    // Compartilhar (remove marca d'água)
    shareBtn.addEventListener("click", () => {
        if (!videoRendered) {
            alert("Renderize o vídeo primeiro!");
            return;
        }
        
        if (!watermarkRemoved) {
            watermarkRemoved = true;
            localStorage.setItem("watermarkRemoved", "true");
            updateUI();
            alert("❤️ OBRIGADO POR COMPARTILHAR! Sua marca d'água foi removida permanentemente. Continue espalhando conhecimento.");
        } else {
            alert("🌟 Você já removeu a marca d'água! Continue compartilhando e ajudando quem precisa.");
        }
        
        // Tentar compartilhar nativamente
        if (navigator.share) {
            navigator.share({
                title: "Meu vídeo no TransVid",
                text: `Aprenda ${currentIdea} de graça!`,
                url: "https://transvid.app"
            }).catch(() => {});
        } else {
            navigator.clipboard.writeText(`TransVid: ${currentIdea} - Vídeo criado gratuitamente!`);
            alert("Link copiado! Compartilhe com seus amigos.");
        }
    });

    // Inicializar
    resetDaily();
    updateUI();
</script>
</body>
</html>