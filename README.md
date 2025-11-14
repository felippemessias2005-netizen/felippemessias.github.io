<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Treinador Esportivo Multi-Abas</title>
    <!-- Carregando Tailwind CSS para Estilização -->
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        /* Definindo a fonte Inter como padrão */
        body { font-family: 'Inter', sans-serif; background-color: #f7f9fb; }
        /* Estilos de transição para o acordeão */
        .accordion-content { 
            transition: max-height 0.4s cubic-bezier(0.25, 0.46, 0.45, 0.94), padding 0.4s cubic-bezier(0.25, 0.46, 0.45, 0.94); 
            max-height: 0; 
            overflow: hidden; 
            padding-top: 0;
            padding-bottom: 0;
        }
        .expanded { 
            max-height: 1500px; /* Suficiente para caber o plano de treino */
            padding-top: 1rem; 
            padding-bottom: 1rem; 
        }
    </style>
</head>
<body class="min-h-screen p-4 md:p-8">

    <div class="max-w-4xl mx-auto">
        <!-- HEADER (Fixo para todas as visualizações, exceto Capa) -->
        <header id="app-header" class="text-center mb-8 hidden">
            <h1 class="text-3xl font-extrabold text-blue-800 tracking-tight">
                🏃 Treinador Digital
            </h1>
            <p class="mt-1 text-md text-gray-500">
                Seu plano de treino personalizado.
            </p>
        </header>

        <!-- Container principal de visualizações -->
        <main id="app-views" class="rounded-xl shadow-2xl bg-white p-6 md:p-10 border border-gray-100">

            <!-- 1. CAPA / VIEW HOME -->
            <section id="view-home" data-view="home" class="view">
                <div class="text-center py-20">
                    <div class="text-7xl mb-6">🏆</div>
                    <h2 class="text-4xl md:text-5xl font-extrabold text-blue-700 mb-4">
                        Bem-vindo ao Treinador AI
                    </h2>
                    <p class="text-xl text-gray-600 mb-8 max-w-lg mx-auto">
                        Seu aplicativo para gerar planos de treinamento específicos e detalhados para o seu esporte.
                    </p>
                    <button onclick="showView('selection')" 
                            class="text-lg font-bold bg-blue-600 hover:bg-blue-700 text-white py-3 px-8 rounded-full 
                                   transition duration-300 transform hover:scale-[1.05] active:scale-[0.98] shadow-xl">
                        Começar o Treino <span class="ml-2">→</span>
                    </button>
                </div>
            </section>

            <!-- 2. INTRODUÇÃO / SELEÇÃO DE ESPORTES -->
            <section id="view-selection" data-view="selection" class="view hidden">
                <h2 class="text-3xl font-bold text-gray-800 mb-6 border-b-2 border-blue-500 pb-2">
                    Escolha o seu Esporte
                </h2>
                <div id="sport-selector" class="grid grid-cols-2 md:grid-cols-3 gap-6">
                    <!-- Os botões de esporte serão injetados aqui via JS -->
                </div>
                
                <div class="mt-8 pt-4 border-t text-right">
                    <button onclick="showView('home')" 
                            class="text-sm font-medium text-gray-600 hover:text-blue-500 transition duration-200">
                        ← Voltar para a Capa
                    </button>
                </div>
            </section>

            <!-- 3. ABAS DE ESPORTES (Dinâmico) -->
            <!-- O conteúdo específico do esporte será renderizado aqui -->
            <section id="view-sport-detail" data-view="sport-detail" class="view hidden">
                <div class="flex items-center justify-between mb-6 border-b-2 border-blue-500 pb-2">
                    <h2 class="text-3xl font-bold text-gray-800">
                        Plano de <span id="current-sport-name"></span>
                    </h2>
                    <button onclick="showView('selection')" 
                            class="text-sm font-medium text-blue-600 hover:text-blue-800 transition duration-200 flex items-center p-2 rounded-lg bg-blue-50 hover:bg-blue-100">
                        <span class="mr-1">←</span> Mudar Esporte
                    </button>
                </div>
                
                <div id="training-details" class="space-y-6">
                    <!-- Detalhes dos 4 treinos para o esporte selecionado -->
                </div>
            </section>

        </main>
    </div>

    <script>
        // Chave e URL para a API do Gemini (deixe a chave vazia)
        const apiKey = ""; 
        const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-09-2025:generateContent?key=${apiKey}`;

        // Estrutura de dados para os esportes e seus 4 tipos de treino (focando nos conceitos solicitados)
        const trainingData = {
            'Futebol': {
                emoji: '⚽',
                trainings: [
                    { type: 'Força e Resistência', description: 'Foco na musculatura das pernas e capacidade de manter o ritmo ao longo do jogo.', prompt: 'Gere um plano de treino de futebol para 90 minutos focado em Força e Resistência. Inclua exercícios de saltos, agachamentos unilaterais e corrida de média intensidade.' },
                    { type: 'Treino de Explosão (Poder)', description: 'Aprimoramento da aceleração e picos de velocidade para quebras de defesa.', prompt: 'Crie um treino de futebol focado em Treino de Explosão para jogadores de campo. Inclua exercícios de aceleração de 0-10 metros e mudança de direção rápida com bola.' },
                    { type: 'Técnico e Tático', description: 'Foco no controle de bola, passe, finalização e posicionamento em campo.', prompt: 'Detalhe um treino de futebol que combine aprimoramento técnico (domínio e passe) com simulação tática em espaço reduzido (4v4).' },
                    { type: 'Recuperação Ativa e Flexibilidade', description: 'Sessão leve para otimizar a recuperação muscular e prevenir lesões.', prompt: 'Elabore uma sessão de treino de futebol para Recuperação Ativa e Flexibilidade. Inclua 30 minutos de alongamento dinâmico e estático e um trote leve de 20 minutos.' }
                ]
            },
            'Natação': {
                emoji: '🏊',
                trainings: [
                    { type: 'Força (Dryland/Musculação)', description: 'Fortalecimento fora d\'água essencial para costas, ombros e core.', prompt: 'Elabore um treino de Força Seca (dryland) para nadadores usando halteres leves ou peso corporal. Foco em rotação do manguito e fortalecimento do core.' },
                    { type: 'Treino de Explosão (Velocidade)', description: 'Foco na potência e velocidade máxima em séries curtas.', prompt: 'Crie um treino de natação para Velocidade/Explosão. Totalize 1.000 metros com séries de 25m e 50m em ritmo máximo com longo intervalo de descanso.' },
                    { type: 'Resistência e Fundo', description: 'Desenvolvimento da capacidade aeróbica e tolerância ao ácido lático.', prompt: 'Gere um treino de natação focado em Resistência, totalizando 2.500 metros. Use o nado Livre com séries longas de 400m e 800m.' },
                    { type: 'Técnica e Eficiência', description: 'Correção de braçada, pernada e otimização da hidrodinâmica.', prompt: 'Detalhe um treino de natação de 1.200 metros focado em Técnica e Eficiência. Inclua drills com prancha, palmar e exercícios de respiração unilateral.' }
                ]
            },
            'Vôlei': {
                emoji: '🏐',
                trainings: [
                    { type: 'Força Funcional e Musculação', description: 'Fortalecimento do tronco (core) e membros superiores (golpe).', prompt: 'Elabore um treino de musculação para jogadores de Vôlei. Foco em fortalecimento do ombro e core, incluindo exercícios como o "press" militar e pranchas laterais.' },
                    { type: 'Treino de Explosão (Pliometria)', description: 'Otimização da impulsão vertical e potência de salto.', prompt: 'Crie um treino de Vôlei focado em Explosão/Pliometria. Inclua exercícios de saltos na caixa, agachamento com salto e trabalho de aterrisagem.' },
                    { type: 'Técnico de Fundamentos', description: 'Aprimoramento de Saque, Manchete, Toque e Ataque.', prompt: 'Gere um plano de treino de Vôlei de 90 minutos focado em aprimoramento Técnico de Fundamentos. Combine exercícios de saque em zona específica e recepção de ataque forte.' },
                    { type: 'Tático de Jogo e Transição', description: 'Simulação de jogo, movimentação de defesa e ataque rápido.', prompt: 'Detalhe um treino tático de Vôlei para simulação de jogo em 6x6. Foco na transição da defesa para o ataque e cobertura de bloqueio.' }
                ]
            },
            'Basquete': {
                emoji: '🏀',
                trainings: [
                    { type: 'Força e Power', description: 'Fortalecimento das pernas, core e membros superiores para contato e arremesso.', prompt: 'Elabore um treino de Força para jogadores de Basquete. Foco em exercícios compostos como Agachamento e Deadlift para aumentar a impulsão e resistência ao contato.' },
                    { type: 'Treino de Explosão (Agilidade)', description: 'Aumento da velocidade de corte, aceleração e mudança de direção.', prompt: 'Crie um treino de Basquete focado em Explosão e Agilidade. Inclua drills de escada de agilidade, cone drills e sprints de 3/4 de quadra.' },
                    { type: 'Musculação Específica (Arremesso)', description: 'Foco na estabilidade e potência do gesto de arremesso.', prompt: 'Gere um treino de Musculação Específica para Basquete. Foco em exercícios de rotação do tronco e fortalecimento de punho e antebraço.' },
                    { type: 'Técnico de Movimentação', description: 'Aprimoramento do Drible, Arremesso e Passe sob pressão.', prompt: 'Detalhe um treino Técnico de Basquete com foco em arremessos em movimento, floaters e passes de penetração, todos executados sob pressão simulada.' }
                ]
            },
            'Corrida': {
                emoji: '👟',
                trainings: [
                    { type: 'Força para Corredores', description: 'Treino de resistência muscular para prevenir lesões e melhorar a postura.', prompt: 'Elabore um treino de Musculação focado em Força para Corredores. Inclua exercícios de core, cadeia posterior e estabilização de joelhos/tornozelos.' },
                    { type: 'Treino de Explosão (Tiros/Pace)', description: 'Aumento da velocidade máxima e melhoria da economia de corrida.', prompt: 'Crie um treino de Corrida de Explosão (Tiros/Intervalado) na pista ou rua. Foco em séries de 200m e 400m em ritmo muito forte, com descanso completo.' },
                    { type: 'Resistência (Longão)', description: 'Construção da base aeróbica e mental para longas distâncias (maratona/meia).', prompt: 'Gere um plano de Longão semanal para corredores de 21km. Foco em ritmo conversacional e hidratação a cada 30 minutos.' },
                    { type: 'Técnica e Cadência', description: 'Foco na postura, frequência de passos e eficiência biomecânica.', prompt: 'Detalhe um treino de Corrida focado em Técnica e Cadência. Inclua educativos (Skipping, Ankle Mobility) e treino com metrônomo para manter a cadência alta.' }
                ]
            },
            'Lutas (Genérico)': {
                emoji: '🥊',
                trainings: [
                    { type: 'Força e Resistência Muscular', description: 'Capacidade de aplicar força repetidamente sem fadiga (clinch, ground-and-pound).', prompt: 'Elabore um treino de Lutas para Força e Resistência Muscular. Inclua exercícios metabólicos com pneus, marretas e sacos de areia (sandbags).' },
                    { type: 'Treino de Explosão (Velocidade de Golpe)', description: 'Aumento da velocidade e potência de socos, chutes ou quedas.', prompt: 'Crie um treino de Lutas focado em Explosão e Velocidade de Golpe. Use séries curtas e rápidas no saco de pancada (bursts) e exercícios pliométricos.' },
                    { type: 'Técnica e Fluidez', description: 'Aprimoramento dos fundamentos, combinações e transições (em pé e no chão).', prompt: 'Gere um plano de treino de Lutas (MMA/Boxe) focado em Técnica e Fluidez. Inclua trabalho de sombra, manopla com foco em combinações de 3 a 5 golpes e drills de transição (queda/chão).' },
                    { type: 'Condicionamento Cardiovascular', description: 'Otimização do cardio para aguentar múltiplos rounds em alta intensidade.', prompt: 'Detalhe um treino de Lutas para Condicionamento Cardiovascular. Use treino intervalado de alta intensidade (HIIT) com corda, bicicleta e simulação de rounds.' }
                ]
            }
        };

        let currentSport = null;

        // --- Funções de Navegação e Acordeão ---

        // Alterna entre as views (abas)
        function showView(viewName, sport = null) {
            document.querySelectorAll('.view').forEach(view => {
                view.classList.add('hidden');
            });
            
            document.getElementById(`view-${viewName}`).classList.remove('hidden');

            // Exibe ou oculta o cabeçalho fixo
            const header = document.getElementById('app-header');
            if (viewName === 'home') {
                header.classList.add('hidden');
            } else {
                header.classList.remove('hidden');
            }

            if (viewName === 'sport-detail' && sport) {
                currentSport = sport;
                renderSportDetails(sport);
            }
        }

        // Renderiza os detalhes dos 4 treinos para o esporte selecionado
        function renderSportDetails(sportName) {
            const detailsContainer = document.getElementById('training-details');
            document.getElementById('current-sport-name').textContent = sportName;
            detailsContainer.innerHTML = '';
            
            const sportData = trainingData[sportName];

            sportData.trainings.forEach((training, index) => {
                const contentId = `content-${sportName.toLowerCase()}-${index}`;
                
                const trainingCard = document.createElement('div');
                trainingCard.className = 'bg-white border border-gray-200 rounded-xl shadow-md overflow-hidden transition duration-300 hover:shadow-lg';
                
                trainingCard.innerHTML = `
                    <div class="p-5 flex justify-between items-center cursor-pointer bg-gray-50 hover:bg-gray-100 rounded-t-xl" 
                         onclick="toggleTrainingDetail('${contentId}', '${training.type}', '${sportName}')">
                        <div>
                            <h3 class="text-xl font-bold text-blue-600">${training.type}</h3>
                            <p class="mt-1 text-sm text-gray-600">${training.description}</p>
                        </div>
                        <div id="icon-${contentId}" class="text-gray-500 transition duration-300 transform">
                            <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="w-6 h-6"><polyline points="6 9 12 15 18 9"></polyline></svg>
                        </div>
                    </div>
                    
                    <div id="${contentId}" data-training-index="${index}" data-loaded="false" class="accordion-content px-5">
                        <div class="p-2 border-t border-gray-200">
                            <!-- Conteúdo gerado pelo LLM será inserido aqui -->
                        </div>
                    </div>
                `;
                detailsContainer.appendChild(trainingCard);
            });
        }

        // --- Funções LLM (API Gemini) ---
        
        // Converte o Markdown para HTML básico para exibição
        const marked = {
            parse: (markdown) => {
                // Apenas um parser muito básico para garantir que o conteúdo seja legível
                let html = markdown
                    .replace(/^### (.*$)/gim, '<h3 class="text-xl font-semibold mt-4 mb-2 text-gray-700">$1</h3>') // H3
                    .replace(/^## (.*$)/gim, '<h2 class="text-2xl font-bold mt-6 mb-3 text-blue-700">$1</h2>') // H2
                    .replace(/^- (.*$)/gim, '<li class="list-disc ml-6 text-gray-700">$1</li>') // List item
                    .replace(/\*\*(.*)\*\*/gim, '<strong>$1</strong>'); // Bold
                
                // Trata as listas (coloca em ul)
                html = html.replace(/<li class="list-disc ml-6/gim, '</ul><ul class="space-y-1 mt-2"><li>').replace(/<\/ul><ul class="space-y-1 mt-2">/gim, '<ul>').replace(/^<li/gim, '<ul class="space-y-1 mt-2"><li').replace(/<\/li>$/gim, '</li></ul>');
                
                // Trata parágrafos
                html = html.split('\n').map(p => {
                    if (p.trim() && !p.startsWith('<h') && !p.startsWith('<ul') && !p.startsWith('</ul') && !p.startsWith('<li>')) {
                        return `<p class="mt-2 text-gray-600">${p.trim()}</p>`;
                    }
                    return p;
                }).join('');

                return html;
            }
        };

        // Função para chamar a API e gerar o plano
        async function loadTrainingPlan(prompt, elementId) {
            document.getElementById(elementId).innerHTML = `
                <div class="flex items-center space-x-3 text-blue-600 py-4">
                    <svg class="animate-spin h-5 w-5" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
                        <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
                        <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
                    </svg>
                    <span class="font-medium">Gerando plano detalhado com AI...</span>
                </div>
            `;

            const systemPrompt = "Você é um treinador esportivo profissional (coach) no Brasil. Sua tarefa é elaborar um plano de treino detalhado e específico (com introdução, 4-5 exercícios principais detalhados e um desaquecimento/volta à calma) para a solicitação do usuário. O plano deve ser motivacional e claro. Use apenas formatação Markdown (títulos, listas, negrito).";
            
            const payload = {
                contents: [{ parts: [{ text: prompt }] }],
                systemInstruction: {
                    parts: [{ text: systemPrompt }]
                },
            };

            const maxRetries = 3;
            let retryCount = 0;
            let resultText = "Não foi possível carregar o plano de treino. Tente novamente ou mude o esporte.";
            
            while (retryCount < maxRetries) {
                try {
                    const response = await fetch(apiUrl, {
                        method: 'POST',
                        headers: { 'Content-Type': 'application/json' },
                        body: JSON.stringify(payload)
                    });

                    if (!response.ok) {
                        throw new Error(`HTTP error! status: ${response.status}`);
                    }

                    const result = await response.json();
                    const candidate = result.candidates?.[0];

                    if (candidate && candidate.content?.parts?.[0]?.text) {
                        resultText = candidate.content.parts[0].text;
                        break; 
                    } else {
                        throw new Error("Resposta da API vazia ou inválida.");
                    }

                } catch (error) {
                    console.error("Erro na chamada da API:", error);
                    retryCount++;
                    if (retryCount < maxRetries) {
                        const delay = Math.pow(2, retryCount) * 1000;
                        await new Promise(resolve => setTimeout(resolve, delay));
                    }
                }
            }
            
            const htmlContent = marked.parse(resultText);
            document.getElementById(elementId).innerHTML = htmlContent;
        }

        // Função para alternar a visibilidade dos detalhes do treino e iniciar a geração
        function toggleTrainingDetail(elementId, trainingType, sportName) {
            const content = document.getElementById(elementId);
            const icon = document.getElementById(`icon-${elementId}`).querySelector('svg');
            
            if (content.classList.contains('expanded')) {
                // Fechar
                content.classList.remove('expanded');
                content.style.maxHeight = '0';
                icon.style.transform = 'rotate(0deg)';
            } else {
                // Abrir
                
                // Fecha todos os outros acordeões
                document.querySelectorAll('.accordion-content.expanded').forEach(openContent => {
                    if (openContent.id !== elementId) {
                        openContent.classList.remove('expanded');
                        openContent.style.maxHeight = '0';
                        document.getElementById(`icon-${openContent.id}`).querySelector('svg').style.transform = 'rotate(0deg)';
                    }
                });
                
                content.classList.add('expanded');
                // Estimativa de altura para transição suave
                content.style.maxHeight = content.scrollHeight + 1500 + 'px'; 
                icon.style.transform = 'rotate(180deg)';
                
                // Carrega o plano de treino via API se ainda não foi carregado
                if (content.dataset.loaded !== 'true') {
                    const trainingIndex = parseInt(content.dataset.trainingIndex);
                    const prompt = trainingData[sportName].trainings[trainingIndex].prompt;

                    loadTrainingPlan(prompt, elementId);
                    content.dataset.loaded = 'true';
                }
            }
        }

        // Inicializa os botões de seleção de esporte ao carregar
        function initSportSelector() {
            const selector = document.getElementById('sport-selector');
            
            Object.keys(trainingData).forEach(sport => {
                const data = trainingData[sport];
                const button = document.createElement('button');
                button.dataset.sport = sport;
                
                // Estilo do botão de esporte
                button.className = 'flex flex-col items-center justify-center p-6 rounded-xl transition duration-300 ease-in-out transform hover:scale-[1.05] active:scale-[0.98] border border-gray-300 bg-white text-gray-800 shadow-md hover:shadow-lg';
                button.setAttribute('onclick', `showView('sport-detail', '${sport}')`);
                
                button.innerHTML = `
                    <span class="text-5xl mb-2">${data.emoji}</span>
                    <span class="font-bold text-center text-lg">${sport}</span>
                `;
                selector.appendChild(button);
            });
            
            // Inicia na capa
            showView('home');
        }

        // Chama a inicialização
        window.onload = initSportSelector;
    </script>
</body>
</html>
