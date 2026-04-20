<!DOCTYPE html>
<html lang="pt">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Contador de Árvores v3</title>
    <script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs@latest/dist/tf.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/@teachablemachine/image@latest/dist/teachablemachine-image.min.js"></script>
    <style>
        body { font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; text-align: center; background: #1a1a1a; color: white; margin: 0; padding: 20px; }
        #webcam-container { margin: 20px auto; border: 4px solid #4CAF50; border-radius: 15px; display: inline-block; overflow: hidden; background: #000; line-height: 0; }
        #counter-box { background: #4CAF50; color: white; padding: 20px; border-radius: 10px; display: inline-block; margin-bottom: 20px; min-width: 200px; }
        .count-number { font-size: 4em; display: block; font-weight: bold; }
        .status { color: #ffeb3b; font-size: 0.9em; margin-bottom: 10px; }
        button { padding: 15px 30px; font-size: 1.2em; background: #2196F3; color: white; border: none; border-radius: 5px; cursor: pointer; }
    </style>
</head>
<body>

    <div id="counter-box">
        <span class="count-number" id="counter">0</span>
        ÁRVORES DETECTADAS
    </div>
    
    <div id="status">A carregar sistema...</div>
    
    <div id="setup-controls">
        <button type="button" onclick="init()">INICIAR CÂMARA</button>
    </div>

    <div id="webcam-container"></div>

<script type="text/javascript">
    // 1. SUBSTITUI PELO TEU LINK (GARANTE QUE TERMINA COM /)
    const URL = "https://teachablemachine.withgoogle.com/models/k6dH4uVFl/";

    let model, webcam, maxPredictions;
    let count = 0;
    let canCount = true;

    // Tentar iniciar automaticamente ao carregar a página
    window.onload = () => {
        setTimeout(init, 1000); 
    };

    async function init() {
        const statusDiv = document.getElementById("status");
        try {
            const modelURL = URL + "model.json";
            const metadataURL = URL + "metadata.json";

            statusDiv.innerText = "A carregar modelo...";
            model = await tmImage.load(modelURL, metadataURL);
            maxPredictions = model.getTotalClasses();

            statusDiv.innerText = "A ligar câmara...";
            
            // Configuração para Telemóvel/Drone (Câmara Traseira)
            const flip = false; 
            webcam = new tmImage.Webcam(400, 400, flip); 
            
            // Tenta forçar câmara traseira
            await webcam.setup({ facingMode: "environment" }); 
            
            await webcam.play();
            statusDiv.innerText = "SISTEMA ATIVO";
            document.getElementById("setup-controls").style.display = "none";
            document.getElementById("webcam-container").appendChild(webcam.canvas);
            
            window.requestAnimationFrame(loop);
        } catch (e) {
            console.error(e);
            statusDiv.innerText = "ERRO: Abre o site via HTTPS ou GitHub Pages!";
            statusDiv.style.color = "red";
        }
    }

    async function loop() {
        webcam.update();
        await predict();
        window.requestAnimationFrame(loop);
    }

    async function predict() {
        const prediction = await model.predict(webcam.canvas);
        
        let foundTreeInThisFrame = false;

        for (let i = 0; i < maxPredictions; i++) {
            // VERIFICA SE O NOME É EXATAMENTE "Arvore"
            if (prediction[i].className === "Arvore" && prediction[i].probability > 0.85) {
                foundTreeInThisFrame = true;
            }
        }

        if (foundTreeInThisFrame) {
            if (canCount) {
                count++;
                document.getElementById("counter").innerText = count;
                canCount = false; // Bloqueia para não contar a mesma árvore 30x por segundo
                console.log("Árvore registada!");
            }
        } else {
            canCount = true; // Só permite contar a próxima quando a anterior sair da vista
        }
    }
</script>
</body>
</html>
