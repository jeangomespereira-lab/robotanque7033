# 🤖 Manual de Programação FRC - Robô Tanque 7033

**Sistema de Tração Diferencial para FIRST Robotics Competition**

---

## 📋 Índice

1. [Estrutura](#estrutura)
2. [Hardware](#hardware)
3. [Guia dos Arquivos](#guia-dos-arquivos)
4. [Como Usar](#como-usar)
5. [Conceitos-Chave](#conceitos-chave)
6. [Dicas & Debugging](#dicas--debugging)

---

## 📁 Estrutura

```
src/java/
├── Robot.java           # Gerencia ciclo de vida e entrada do operador
└── DriveTrain.java      # Sistema de tração diferencial
```

---

## ⚙️ Hardware

| Componente | Detalhes |
|---|---|
| **Motores Líderes** | 2x Talon FX (portas CAN 1 e 3) |
| **Motores Seguidores** | 1x Talon FX (porta 2) + 1x Victor SPX (porta 4) |
| **Encoders** | 2x Quadratura (pinos DIO 0-1 e 2-3) - medem distância |
| **IMU** | Integrada no roboRIO - rastreia ângulo |
| **Controle** | Joystick/Gamepad na Driver Station |

---

## 📖 Guia dos Arquivos

### Robot.java - Orquestrador Principal

**O que faz**: Lê o joystick do operador e comanda o robô

**Fluxo**:
1. Lê joystick esquerdo Y (velocidade) e direito X (rotação)
2. Aplica `SlewRateLimiter` para suavizar (evita movimentos bruscos)
3. Escala para valores reais (m/s e rad/s)
4. Envia para `drive.drive(velocidade, rotação)`

**Entrada do Operador**:
- Joystick esquerdo: movimento linear (-1 a 1)
- Joystick direito: rotação (-1 a 1)

### DriveTrain.java - Sistema de Tração

**O que faz**: Converte comandos em voltagem para os motores

**Responsabilidades**:
- **Tração Diferencial**: Controla cada lado independentemente
  - Reto: ambas rodas mesma velocidade
  - Girar: rodas velocidades opostas
- **Cinemática**: Converte (velocidade_linear, velocidade_angular) em velocidades das rodas
- **Controle**: PID + Feedforward ajustam voltagem para atingir velocidade exata
- **Odometria**: Rastreia posição (x, y, ângulo) do robô

**Constantes importantes**:
```java
kMaxVelocity = 3.0 m/s              // Velocidade máxima
kMaxAngularVelocity = 2π rad/s      // 1 rotação/segundo
kTrackwidth = 0.762 m               // Distância entre rodas
kWheelRadius = 0.0508 m             // Raio de cada roda
kEncoderResolution = 4096           // Pulsos por volta
```

**Por que isso importa**:
- `kTrackwidth` define como a rotação se transforma em diferença de velocidade entre rodas.
- `kWheelRadius` garante que os encoders retornem distância em metros, não apenas pulsos.
- `kEncoderResolution` deve refletir o encoder real instalado para odometria precisa.

---

## 🔄 Fluxo de Dados

1. O operador move o joystick
2. `Robot.java` lê os valores e aplica suavização
3. Valores são escalados para unidade do robô (m/s, rad/s)
4. `DriveTrain.drive()` converte em velocidades das rodas
5. `setVelocities()` calcula voltagem com PID + feedforward
6. Motores recebem voltagem e rodam
7. Encoders e IMU atualizam o estado do robô

---

## 🎮 Como Usar

### Teleoperado (Controle Manual)
- Joystick esquerdo para frente/trás: move linear
- Joystick direito esquerda/direita: rotação
- Combine para movimentos complexos

### Autônomo
O robô segue programa pré-definido usando odometria:
- Mover distância X
- Girar ângulo Y
- Navegar trajetórias

---

## 💡 Conceitos-Chave

### Tração Diferencial (Tank Drive)
Cada lado (esquerdo/direito) pode girar independentemente:
- **Mesmo velocidade** → movimento reto
- **Velocidades opostas** → gira no lugar
- **Velocidades diferentes** → movimento curvo

### Por que PID + Feedforward?
- **Feedforward**: Tensão inicial aproximada (rápido)
- **PID**: Ajusta baseado no erro (preciso)
- Juntos: robô atinge velocidade exata rapidamente

### Por que Encoders e IMU?
- **Encoders**: Medem distância de cada roda → controle preciso
- **IMU**: Rastreia ângulo → saber orientação do robô
- **Odometria**: Estima posição (x, y, θ) combinando os dois

### Limitadores de Taxa (SlewRateLimiter)
Suaviza mudanças de entrada (0 a 100% em 3 segundos):
- Evita stress mecânico
- Reduz picos de corrente
- Melhora controle e durabilidade

---

## 🚀 Dicas & Debugging

### Antes de Começar
1. **Entenda hardware**: Quantos motores? Em quais portas?
2. **Verifique inversões**: Se um lado move errado, inverta com `.setInverted(true)`
3. **Calibre PID**: Ganhos padrão são (1, 0, 0) - ajuste para seu robô

### Modificações Comuns
```java
// Aumentar velocidade
kMaxVelocity = 5.0;  // de 3.0 a 5.0 m/s

// Mais sensibilidade
velocityLimiter = new SlewRateLimiter(5);  // de 3 a 5

// Rotação mais rápida
kMaxAngularVelocity = 4 * Math.PI;  // 2 voltas/segundo
```

### Troubleshooting

| Problema | Solução |
|---|---|
| Motor não se move | Verifique porta CAN, inversão, bateria |
| Lado mais rápido | Ajuste Kp do PID do outro lado |
| Oscilação/vibração | Reduza Kp do PID |
| Odometria errada | Resete encoders, verifique resolução |
| Entrada errada | Use `println()` para debugar joystick |

### Próximos Passos
- Adicione subsistemas (braço, garra, etc.)
- Implemente trajetórias suave (`TrajectoryGenerator`)
- Integre câmera para visão
- Teste outros modos de controle (arcade, curvature)

---

## � Notas de Equipe

- **Organize o código por subsistema**: mantenha `Drivetrain` separado de outros mecanismos futuros.
- **Documente alterações no CAN ID** sempre que mudar a fiação dos motores.
- **Testes no chão são essenciais**: faça runs em baixa velocidade antes de subir para o campo.
- **Calibração no início de cada sessão**: resetar encoders e IMU antes de operar.

---

## �📚 Referências
- **WPILib**: https://docs.wpilib.org/
- **CTRE Phoenix**: https://docs.ctre-phoenix.com/
- **Chief Delphi**: https://www.chiefdelphi.com/
