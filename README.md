# Tutor de Cubo Mágico com Visão Computacional

Este projeto tem como objetivo criar um tutor de cubo mágico usando visão computacional. A aplicação usa uma webcam para reconhecer um cubo mágico 3x3, identificar seu estado atual, calcular uma solução através do método de Kociemba e guiar o usuário passo a passo até a resolução.

A proposta combina captura de vídeo, processamento de imagem, classificação de cores, validação lógica do cubo, algoritmos clássicos de resolução e uma interface interativa para acompanhamento do usuário.

## Visão Geral

O fluxo principal do sistema é:

```text
Webcam
  -> Detecção da face do cubo
  -> Leitura das cores
  -> Montagem do estado lógico do cubo
  -> Validação do estado
  -> Solver
  -> Tutorial passo a passo
  -> Verificação dos movimentos do usuário
```

A ideia original do projeto é um MVP que funciona como guia onde o usuário mostra as seis faces do cubo uma por vez. Depois, ele pode evoluir para um tutor em tempo real, com verificação automática de movimentos e instruções visuais sobre a imagem da câmera.

## Objetivo Do MVP

O primeiro objetivo é criar uma versão funcional e confiável:

> O usuário mostra as seis faces do cubo, o sistema identifica as cores, monta o estado completo e gera uma solução.

Nesta primeira versão, não é necessário acompanhar cada movimento ao vivo. O foco é resolver corretamente o cubo a partir das imagens capturadas.

Funcionalidades do MVP:

- Abrir a webcam.
- Exibir uma grade fixa 3x3 para alinhamento da face.
- Capturar as cores dos nove quadrados da face.
- Repetir o processo para as seis faces.
- Calibrar as cores com base na iluminação atual.
- Montar a representação lógica do cubo.
- Validar se o estado é plausível.
- Calcular a solução usando um solver.
- Exibir a sequência de movimentos.
- Traduzir os movimentos para instruções em português.

## Stack Recomendada

Para o MVP:

```text
Python
OpenCV
NumPy
kociemba
Streamlit
pytest
```

Para uma versão mais completa:

```text
Frontend: React ou Next.js
Backend: FastAPI
Computer Vision: OpenCV, com possível uso futuro de YOLO
Solver: kociemba ou cubing.js
Banco opcional: SQLite
Deploy inicial: local
```

## Fase 1: Definir O Escopo

### Intenção

O projeto deve primeiro provar que consegue ler um cubo e gerar uma solução.

### Implementação

Definir o fluxo inicial:

1. Abrir webcam.
2. Pedir uma face ao usuário.
3. Capturar a face.
4. Detectar as nove cores.
5. Repetir até ter as seis faces.
6. Montar a string do cubo.
7. Resolver com um algoritmo pronto.
8. Mostrar a solução.

O acompanhamento ao vivo deve ficar para uma fase posterior.

## Fase 2: Captura De Vídeo Pela Webcam

### Intenção

Garantir que a aplicação consegue acessar a webcam e capturar imagens em tempo real.

### Implementação

Usar OpenCV para abrir a câmera:

```python
import cv2

cap = cv2.VideoCapture(0)

while True:
    ret, frame = cap.read()
    if not ret:
        break

    cv2.imshow("Webcam", frame)

    key = cv2.waitKey(1)
    if key == ord("c"):
        cv2.imwrite("face_capture.png", frame)
    elif key == ord("q"):
        break

cap.release()
cv2.destroyAllWindows()
```

Resultado esperado:

- A webcam abre corretamente.
- O usuário consegue ver o vídeo ao vivo.
- É possível capturar uma imagem pressionando uma tecla.

## Fase 3: Guiar O Usuário Para Mostrar Cada Face

### Intenção

Controlar a orientação do cubo e reduzir erros na montagem do estado final.

### Implementação

Criar um fluxo guiado em que o sistema pede as faces em uma ordem definida. Uma opção é capturar por posição:

1. Frente
2. Direita
3. Trás
4. Esquerda
5. Cima
6. Baixo

Outra possível opção, capturar por cor central:

1. Branco
2. Vermelho
3. Azul
4. Laranja
5. Verde
6. Amarelo

Para evitar ambiguidade, o sistema pode orientar o usuário com frases como:

```text
Mostre a face branca mantendo a face verde para cima.
Agora gire o cubo para mostrar a face vermelha.
```

Resultado esperado:

- O sistema sabe qual face está sendo capturada.
- A orientação relativa das faces fica consistente.
- O risco de montar um cubo inválido diminui.

## Fase 4: Detectar A Face Do Cubo

### Intenção

Ler os nove quadrados visíveis de uma face do cubo.

### Implementação Recomendada Para O MVP

Começar com uma grade fixa na tela. O usuário alinha a face do cubo dentro dessa grade, e o sistema coleta as cores em nove regiões predefinidas.

Exemplo visual:

```text
+---+---+---+
|   |   |   |
+---+---+---+
|   |   |   |
+---+---+---+
|   |   |   |
+---+---+---+
```

Cada célula da grade possui um ponto central. O sistema coleta uma pequena região ao redor desse ponto e calcula a cor média.

Exemplo:

```python
patch = frame[y - 10:y + 10, x - 10:x + 10]
mean_color = patch.mean(axis=(0, 1))
```

Resultado esperado:

- O sistema extrai nove amostras de cor da face.
- Cada amostra representa um quadrado do cubo.

## Fase 5: Classificar As Cores

### Intenção

Converter os valores capturados pela câmera em uma das seis cores do cubo:

```text
branco, amarelo, vermelho, laranja, azul, verde
```

### Desafios

A leitura de cor pode mudar conforme:

- Iluminação do ambiente.
- Sombra.
- Reflexos.
- Qualidade da webcam.
- Distância do cubo.
- Balanço de branco automático da câmera.

### Implementação

Evitar comparar cores apenas em RGB ou BGR. Usar espaços como HSV ou LAB.

Com OpenCV:

```python
hsv = cv2.cvtColor(frame, cv2.COLOR_BGR2HSV)
```

Possivelmente uma estratégia mais robusta é fazer calibração por sessão:

1. Capturar as seis faces.
2. Usar o quadrado central de cada face como referência daquela cor.
3. Classificar os demais quadrados pela menor distância até uma referência conhecida.

Exemplo conceitual:

```python
import numpy as np

def classify_color(sample, reference_colors):
    best_color = None
    best_distance = float("inf")

    for color_name, ref in reference_colors.items():
        distance = np.linalg.norm(sample - ref)
        if distance < best_distance:
            best_distance = distance
            best_color = color_name

    return best_color
```

Resultado esperado:

```python
[
    ["white", "white", "red"],
    ["green", "white", "blue"],
    ["yellow", "orange", "blue"]
]
```

## Fase 6: Montar O Estado Lógico Do Cubo

### Intenção

Transformar as seis faces capturadas em uma representação compatível com um solver.

Muitos solvers usam a notação:

```text
U R F D L B
```

Onde:

- `U`: Up, face de cima.
- `R`: Right, face direita.
- `F`: Front, face da frente.
- `D`: Down, face de baixo.
- `L`: Left, face esquerda.
- `B`: Back, face de trás.

Para esse caso o solver de Kociemba parece uma boa opçãp e normalmente espera uma string de 54 caracteres:

```text
UUUUUUUUURRRRRRRRRFFFFFFFFFDDDDDDDDDLLLLLLLLLBBBBBBBBB
```

### Implementação

Criar um mapeamento entre cores e faces:

```python
color_to_face = {
    "white": "U",
    "red": "R",
    "green": "F",
    "yellow": "D",
    "orange": "L",
    "blue": "B",
}
```

Esse mapeamento depende da convenção escolhida no fluxo de captura.

Resultado esperado:

- Uma string válida de 54 caracteres.
- Um estado lógico compatível com o solver.

## Fase 7: Validar O Cubo

### Intenção

Evitar que erros de leitura gerem um estado impossível.

### Validações Básicas

- Existem exatamente nove quadrados de cada cor.
- Cada face possui um centro definido.
- As cores opostas seguem o padrão esperado.
- A string possui 54 posições.
- O solver aceita o estado.

Exemplo:

```python
from collections import Counter

def validate_counts(cube_string):
    counts = Counter(cube_string)

    for face in "URFDLB":
        if counts[face] != 9:
            raise ValueError(
                f"Face {face} tem {counts[face]} posições, esperado 9"
            )
```

Resultado esperado:

- O sistema detecta leituras inconsistentes.
- O usuário pode recapturar uma face problemática.

## Fase 8: Resolver O Cubo

### Intenção

Gerar uma sequência de movimentos que resolva o cubo.

### Implementação

Usar uma biblioteca pronta, como `kociemba`:

```python
import kociemba

solution = kociemba.solve(cube_string)
print(solution)
```

Exemplo de saída:

```text
R U R' U' F2 D L'
```

Resultado esperado:

- O sistema retorna uma solução válida.
- A solução pode ser exibida como lista de movimentos.

## Fase 9: Traduzir Movimentos Para Instruções Humanas

### Intenção

Transformar a notação técnica em instruções compreensíveis para iniciantes.

Exemplos:

- `R`: gire a face direita uma vez no sentido horário.
- `R'`: gire a face direita uma vez no sentido anti-horário.
- `R2`: gire a face direita duas vezes.

Exemplo de implementação:

```python
MOVE_DESCRIPTIONS = {
    "R": "Gire a face direita no sentido horário.",
    "R'": "Gire a face direita no sentido anti-horário.",
    "R2": "Gire a face direita duas vezes.",
    "L": "Gire a face esquerda no sentido horário.",
    "L'": "Gire a face esquerda no sentido anti-horário.",
    "U": "Gire a face de cima no sentido horário.",
    "U'": "Gire a face de cima no sentido anti-horário.",
}
```

Resultado esperado:

- O usuário recebe instruções claras.
- A aplicação começa a funcionar como um tutor, não apenas como um resolvedor.

## Fase 10: Criar A Interface Do Tutor

### Intenção

Transformar a lógica em uma experiência usável.

### Elementos Da Interface

- Vídeo da webcam ao vivo.
- Grade de captura da face.
- Botão para capturar face.
- Indicação da próxima face.
- Prévia das cores detectadas.
- Botões para confirmar ou recapturar.
- Lista de movimentos.
- Modo passo a passo.
- Botão para avançar o movimento.
- Ilustração da rotação atual.

### Tecnologias Possíveis

Para protótipo rápido:

- Streamlit.
- Gradio.
- Flask.
- PyQt.

Para aplicação mais profissional:

- React.
- Next.js.
- FastAPI.
- WebRTC.

Recomendação inicial:

```text
Python + OpenCV + Streamlit
```

## Fase 11: Verificar Se O Usuário Fez O Movimento Correto

### Intenção

Comparar o movimento esperado com o estado observado após o usuário executar uma instrução.

### Implementação

O sistema possui um estado atual:

```python
current_state
```

Também possui o próximo movimento esperado:

```python
expected_move = "R"
```

Ele pode simular esse movimento:

```python
expected_state = apply_move(current_state, expected_move)
```

Depois, lê novamente o cubo e compara:

```python
observed_state == expected_state
```

Para uma primeira versão, a verificação pode ser manual:

- O usuário faz o movimento.
- Clica em "feito".
- O app avança para o próximo passo.

Depois, pode evoluir para:

- Verificação parcial de uma face.
- Verificação automática por câmera.
- Acompanhamento contínuo em tempo real.

## Fase 12: Overlay Visual Com Setas

### Intenção

Mostrar diretamente na imagem da webcam qual face deve ser girada e em qual direção.

### Implementação

Com OpenCV:

```python
cv2.arrowedLine(frame, start_point, end_point, color, thickness)
```

Ou em uma interface web:

- Usar um elemento `canvas`.
- Desenhar setas sobre o vídeo.
- Destacar a região da face que será girada.

Para o movimento `R`, por exemplo:

- Destacar a coluna direita da face visível.
- Mostrar seta indicando rotação.
- Exibir a notação `R`.

Desafio:

- O overlay só é confiável se o sistema souber a orientação atual do cubo.

Para o MVP, uma ilustração estática ao lado da câmera é mais simples e confiável.

## Fase 13: Melhorar A Detecção Automática

### Intenção

Permitir que o sistema encontre o cubo automaticamente, sem depender de uma grade fixa.

### Abordagem Com OpenCV

Pipeline possível:

```text
Imagem
  -> Conversão para escala de cinza
  -> Blur
  -> Canny
  -> Detecção de contornos
  -> Filtro de quadriláteros
  -> Agrupamento de nove quadrados
  -> Leitura de cores
```

Técnicas úteis:

- Thresholding.
- Detecção de bordas.
- Contornos.
- Correção de perspectiva.
- Agrupamento espacial.

### Abordagem Com Modelo Neural

Possibilidades:

- YOLO.
- Detectron2.
- TensorFlow Lite.
- Modelo customizado para detectar stickers.

Recomendação:

- Começar com OpenCV.
- Usar IA apenas depois que o fluxo básico estiver funcionando.

## Fase 14: Construir Um Modelo Interno Do Cubo

### Intenção

Representar o cubo dentro do código para simular movimentos, validar estados e acompanhar o progresso.

### Implementação

Criar uma classe:

```python
class Cube:
    def __init__(self, state):
        self.state = state

    def apply_move(self, move):
        pass

    def is_solved(self):
        pass

    def to_solver_string(self):
        pass
```

Se possível, usar uma biblioteca pronta para simular movimentos, porque implementar rotações corretamente pode gerar bugs sutis.

Resultado esperado:

- O sistema sabe como o cubo deve ficar após cada movimento.
- O tutor consegue acompanhar o progresso do usuário.

## Fase 15: Criar Modos De Ensino

### Intenção

Fazer o projeto ir além de resolver o cubo e se tornar uma ferramenta de aprendizado.

### Modos Possíveis

#### Modo Solução Rápida

O app calcula a solução e guia o usuário pelos movimentos.

#### Modo Iniciante

Resolve por etapas:

1. Cruz branca.
2. Cantos brancos.
3. Segunda camada.
4. Cruz amarela.
5. Orientação da face amarela.
6. Permutação dos cantos.
7. Permutação das arestas.

#### Modo Treino

O usuário pratica algoritmos específicos, como:

```text
R U R' U'
```

#### Modo Explicativo

O app explica a intenção de cada etapa:

```text
Agora vamos formar a cruz branca sem destruir os cantos já resolvidos.
```

## Arquitetura Recomendada

Estrutura proposta:

```text
rubiks_tutor/
│
├── app/
│   ├── main.py
│   ├── camera.py
│   └── ui.py
│
├── vision/
│   ├── grid_reader.py
│   ├── color_classifier.py
│   ├── calibration.py
│   └── cube_detector.py
│
├── cube/
│   ├── state.py
│   ├── notation.py
│   ├── validator.py
│   └── moves.py
│
├── solver/
│   └── kociemba_solver.py
│
├── tutor/
│   ├── instructions.py
│   ├── step_manager.py
│   └── feedback.py
│
├── assets/
│   └── move_diagrams/
│
├── tests/
│   ├── test_cube_state.py
│   ├── test_color_classifier.py
│   └── test_solver_format.py
│
└── README.md
```

### Responsabilidades Dos Módulos

`camera.py`

- Abrir webcam.
- Capturar frames.
- Controlar resolução.

`grid_reader.py`

- Desenhar grade.
- Definir pontos de amostragem.
- Extrair cores médias.

`color_classifier.py`

- Converter cores.
- Comparar com referências.
- Classificar cada sticker.

`calibration.py`

- Registrar as cores centrais.
- Ajustar leitura para a iluminação atual.

`state.py`

- Representar o cubo.
- Armazenar faces.
- Exportar estado.

`validator.py`

- Verificar contagem de cores.
- Checar consistência básica.

`kociemba_solver.py`

- Converter estado para formato do solver.
- Chamar o solver.
- Retornar movimentos.

`instructions.py`

- Traduzir notação em linguagem humana.

`step_manager.py`

- Controlar passo atual.
- Avançar e voltar.
- Guardar progresso.

## Ordem Real De Implementação

1. Criar estrutura do projeto.
2. Abrir webcam com OpenCV.
3. Desenhar grade fixa 3x3.
4. Capturar a cor média dos nove pontos.
5. Mostrar a face detectada em texto ou matriz.
6. Criar calibração das seis cores.
7. Capturar as seis faces.
8. Montar string do cubo.
9. Validar contagem das cores.
10. Integrar solver.
11. Mostrar solução textual.
12. Criar interface mais amigável.
13. Adicionar confirmação e recaptura de face.
14. Adicionar tutorial passo a passo.
15. Adicionar ilustrações dos movimentos.
16. Adicionar verificação após movimento.
17. Melhorar detecção automática.
18. Evoluir para overlay em tempo real.

## Principais Riscos Técnicos

### Iluminação

A mesma cor pode parecer diferente dependendo da luz.

Soluções possíveis:

- Calibração por sessão.
- Uso de HSV ou LAB.
- Evitar sombras.
- Calcular média de uma região, não de um único pixel.

### Reflexo

Cubos brilhantes podem atrapalhar a leitura.

Soluções possíveis:

- Detectar regiões muito saturadas.
- Pedir nova captura.
- Usar iluminação difusa.

### Orientação Do Cubo

O sistema pode confundir cima, baixo, esquerda e direita.

Soluções possíveis:

- Usar fluxo guiado.
- Pedir uma face de referência para cima.
- Usar centros como âncoras.

### Estado Impossível

Uma leitura errada pode gerar um cubo que não existe fisicamente.

Soluções possíveis:

- Validar antes de chamar o solver.
- Mostrar prévia para o usuário corrigir.
- Permitir edição manual das cores.

### Instruções Ambíguas

Comandos como "gire para a direita" podem ser confusos.

Soluções possíveis:

- Usar notação padrão.
- Mostrar diagrama.
- Explicar o ponto de vista de cada movimento.

## Cronograma Sugerido

### Semana 1

- Webcam.
- Grade fixa.
- Captura de cores.
- Testes de classificação.

### Semana 2

- Calibração.
- Captura das seis faces.
- Validação do estado.

### Semana 3

- Integração com solver.
- Exibição da solução.
- Interface simples.

### Semana 4

- Tutorial passo a passo.
- Correção manual de cores.
- Melhorias de usabilidade.

### Depois Do MVP

- Overlay visual.
- Verificação automática de movimentos.
- Detecção automática do cubo.
- Modo tutor em tempo real.
- Modo iniciante.
- Modo treino.
- Histórico de erros.
- Estatísticas de tempo.
- Suporte mobile.

## Versão Avançada

Após o MVP, o projeto pode evoluir para:

- Detecção automática do cubo.
- Overlay com setas na webcam.
- Correção automática dos movimentos.
- Narração por voz.
- Modo iniciante.
- Modo treino.
- Histórico de tentativas.
- Estatísticas de resolução.
- Suporte a dispositivos móveis.
- Modelo treinado para detectar stickers.
- Interface web completa.

## Resumo Da Estratégia

O caminho mais eficiente para construir este projeto é:

1. Começar sem deep learning.
2. Usar OpenCV com uma grade guiada.
3. Resolver o cubo com um algoritmo pronto.
4. Focar primeiro na leitura confiável das cores.
5. Evoluir depois para acompanhamento em tempo real.

## Hashtags

#ComputerVision #VisaoComputacional #OpenCV #Python #MachineLearning #ArtificialIntelligence #RubiksCube #CuboMagico #ImageProcessing #ProcessamentoDeImagem #Algoritmos #Kociemba #ProjetoDePortfolio #PortfolioDev #ComputerScience 
