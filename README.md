# ENTERPRISE-CHALLENGE-2026-GOODWE-FIAP

Markdown# ⚡ EV ChargeOps — Relatório de Integração Final
**Consolidação Arquitetural, Regulatória, Inteligência Artificial e Engenharia de Rateio**

* **Responsável:** Integrador & Tech Lead
* **Escopo:** Visão Consolidada de Frentes
* **Data:** Junho de 2026

---

## 1. Introdução e Alinhamento Estratégico

Este documento consolida o mapeamento conceitual, regulatório, operacional e técnico da plataforma EV ChargeOps. Como integrador do projeto, este relatório unifica as visões de infraestrutura física, conformidade de mercado, arquitetura de software e inteligência computacional. O objetivo final é estruturar um ecossistema escalável para a gestão de recargas de Veículos Elétricos (VEs) em ambientes compartilhados, provendo clareza técnica e viabilidade comercial para a próxima sprint.

## 2. Contexto, Desafios Operacionais e Infraestrutura Física

A eletrificação da frota veicular nacional traz desafios imediatos para a gestão condominial e corporativa. O projeto mapeia e responde aos seguintes pilares críticos:

* **Segurança e Adequação Elétrica:** A infraestrutura requer conformidade estrita com as diretrizes ABNT NBR 5410, NBR 17019, NBR IEC 61851-1 e com as regulamentações estaduais de prevenção a incêndios (Portarias CCB-009/800/2025 e CCB-008/800/2025 do Corpo de Bombeiros de SP).
* **Captura de Dados Transacionais:** Durante o processo de recarga, o sistema atua capturando variáveis do carregador, incluindo a identificação do usuário, horários de início e término, volume de energia entregue em kWh, curva de potência utilitária, tempo total de conexão e o status operacional do hardware.

## 3. Base Regulatória e Conectividade de Hardware

A arquitetura opera em conformidade legal com o mercado elétrico brasileiro, mitigando vulnerabilidades de aprisionamento tecnológico (*Vendor Lock-in*):

### 3.1. Resolução Normativa ANEEL nº 1.000/2021
A resolução assegura a comercialização de serviços de recarga por entes privados, estabelecendo a necessidade de comunicação formal prévia à concessionária de energia e a obrigatoriedade do uso de protocolos de comunicação abertos (OCPP baseado em mensagens JSON) para dispositivos de acesso público ou compartilhado.

### 3.2. Interfaces de Comunicação da Borda: GoodWe HCA G2
O hardware de carregamento GoodWe HCA G2 atua na borda da solução. Suas interfaces de conectividade física e lógica são descritas abaixo:

| Interface | Camada OSI | Função no Ecossistema EV ChargeOps |
| :--- | :--- | :--- |
| **RFID** | Física / Enlace | **Autenticação Local:** Identifica e associa de forma segura a sessão física de recarga ao cadastro do morador. |
| **Bluetooth** | Rede (PAN) | **Comissionamento:** Canal exclusivo do técnico de campo para a parametrização e inserção de credenciais de rede local. |
| **Wi-Fi / LAN** | Rede (WLAN) | **Conexão Cloud:** Canal principal de tráfego TCP/IP para envio de dados telemétricos aos servidores do sistema. |
| **RS-485** | Física (Serial) | **Integração M2M:** Comunicação serial via Modbus RTU com medidores inteligentes (*Smart Meters*) para balanceamento dinâmico de carga. |

### 3.3. Serviços e APIs Complementares de Mercado
Para elevar a inteligência de negócios, o sistema acopla dados de fontes externas em seu ecossistema:

* **Google Places API (`evChargeOptions`):** Consome payloads detalhados de estações ao redor, mapeando tipos de conectores (Tipo 2, CCS2) e potências operacionais para inteligência tarifária.
* **Open Charge Map API:** Integra endpoints RESTful via requisições GET para analisar mapas de uso regionais, permitindo futuras estratégias de abertura dos pontos para monetização de recargas de visitantes.

## 4. Arquitetura de Software e Fluxo de Dados End-to-End

A plataforma EV ChargeOps é estruturada em uma topologia de quatro macrocamadas para processar e expor as informações coletadas:

1. **Camada Física:** Composta pelo carregador de borda GoodWe HCA G2 e o veículo conectado, responsável pela originação dos dados elétricos e de identificação.
2. **Camada de Conectividade:** Garante o transporte seguro dos pacotes. O fluxo segue a rota: Carregador → Rede Local (Wi-Fi/LAN) → Servidor Central EV ChargeOps → Banco de Dados, aplicando autenticação nas APIs REST.
3. **Camada de Aplicação (Back-end e IA):** Centraliza as regras de negócio, persistência de sessões, motor de cálculo financeiro e algoritmos de Machine Learning.
4. **Camada de Apresentação (Interfaces):** Dividida entre a Área do Usuário (acompanhamento de faturas, consumo e alertas) e a Área do Gestor (administração de vagas, auditoria de consumo geral e monitoramento de falhas).

## 5. Diagrama Topológico da Solução

Conforme mapeado na engenharia de sistemas do ecossistema, o fluxo estrutural de dados e dependências lógicas segue a topologia ilustrada abaixo:

```text
[ Usuário / Veículo Elétrico ]
  (Origem da Demanda Física)
              ↓
      [ GoodWe HCA G2 ]
 (Estação de Recarga - Borda)
              ↓
       [ Conectividade ]
(Wi-Fi / LAN / Bluetooth / RFID)
              ↓
          [ Back-end ]
(Aplicação / Orquestração Central)
         ↙            ↘
[ Banco de Dados ]   [ Regras de Negócio ]
[ APIs / Serviços ]  [ IA / Machine Learning ]
         ↘            ↙
[ Dashboard Gestor ] [ Aplicativo Usuário ]

## 6. Regras de Negócio e Engenharia do Modelo de Rateio

O modelo financeiro central baseia-se no Modelo Medido (Cobrança Individualizada), garantindo que cada usuário responda proporcionalmente ao seu consumo elétrico real[cite: 1].

### 6.1. Formulação Tarifária e Estrutura Financeira

A cobrança mensal consolidada para cada conta de morador é regida pela equação polinomial híbrida[cite: 1]:

$$V_{Total} = \sum (kWh_{consumido} \times Tarifa_{kWh}) + F_{Manutencao} + T_{Ociosidade}$$

O sistema processa o consumo individual de forma transparente[cite: 1]. Por exemplo, se uma unidade registra o consumo de **35 kWh** no ciclo com uma tarifa base estabelecida em **R$ 0,90/kWh**, o sistema realiza a operação direta calculando **R$ 31,50** de consumo energético puro, acrescido das taxas de contorno aplicáveis[cite: 1].

### 6.2. Matriz de Tratamento de Exceções e Contorno

* **Unidades Não Usuárias:** Moradores sem veículo elétrico cadastrado ficam isentos do consumo energético e do custeio operacional direto do software, incidindo apenas em taxas fixas de infraestrutura predial caso determinado pela convenção[cite: 1].
* **Sessões Interrompidas ou Falhas de Rede:** Em casos de desconexão abrupta, a lógica do back-end encerra a sessão com os últimos dados consistentes recebidos, tarifando estritamente a energia (kWh) que foi efetivamente entregue à bateria do veículo[cite: 1].
* **Múltiplos Veículos por Unidade Residencial:** O sistema suporta o vínculo de múltiplos identificadores de hardware (cartões RFID ou subcontas) sob a mesma unidade habitacional[cite: 1]. Os consumos são discriminados por usuário/veículo e unificados em uma única fatura mensal consolidada da unidade[cite: 1].
* **Gestão de Ociosidade em Vagas Comuns:** Ao detectar carga a **100%**, o sistema inicia uma contagem de tolerância de 30 minutos combinada a alertas push[cite: 1]. Excedido o período, adiciona-se o valor de $T_{Ociosidade}$ por tempo de retenção da vaga[cite: 1].

## 7. Camada de Inteligência Artificial

A Inteligência Artificial atua de forma nativa na camada de aplicação através de duas frentes especializadas[cite: 1]:

### 7.1. Detecção de Anomalias nas Sessões

Utilizando modelos de Machine Learning aplicados ao histórico transacional (horários, curvas de kWh e tempos de recarga), o módulo identifica consumos fora do padrão preditivo, sessões retidas indevidamente ou indícios de falhas de isolamento e degradação do carregador[cite: 1]. O impacto direto é a redução de perdas energéticas e mitigação de riscos operacionais[cite: 1].

### 7.2. Previsão de Consumo Energético

Através de modelos de regressão e análise de séries temporais sobre o histórico de utilização coletado, a IA projeta a demanda de carga futura do condomínio[cite: 1]. Isso viabiliza o planejamento preventivo para evitar sobrecargas na subestação do edifício e fornece inteligência para estratégias de gerenciamento de demanda (*Peak Shaving*)[cite: 1].

## 8. Planejamento do Banco de Dados Inicial

Para suportar as operações, a modelagem de dados relacional inicial conta com as seguintes entidades principais[cite: 1]:

| Entidade | Atributo Base | Tipo / Descrição |
| :--- | :--- | :--- |
| **Usuário** | `id_usuario` | **Primary Key** (Identificador único do morador)[cite: 1] |
| | `nome` / `contato` | **Varchar** (Dados cadastrais e de comunicação)[cite: 1] |
| | `unidade` | **Varchar** (Apartamento, bloco ou identificação de lote)[cite: 1] |
| **Sessão de Recarga** | `id_sessao` | **Primary Key** (Identificador único da transação)[cite: 1] |
| | `id_usuario` | **Foreign Key** (Vínculo com a entidade Usuário)[cite: 1] |
| | `inicio` / `fim` | **Timestamp** (Registro temporal de conexão e desconexão)[cite: 1] |
| | `energia_kwh` | **Decimal** (Métrica de consumo coletada da borda)[cite: 1] |
| **Fatura** | `id_fatura` | **Primary Key** (Identificador financeiro)[cite: 1] |
| | `consumo` | **Decimal** (Acumulado de kWh do período de apuração)[cite: 1] |
| | `valor_total` | **Decimal** (Resultado do motor de cálculo pós-exceções)[cite: 1] |

## 9. Plano de Ação e Roadmap Tecnológico: Sprint 2

O plano de ação para a Sprint 2 consolida os requisitos em um cronograma de desenvolvimento focado na criação de um protótipo funcional integrado[cite: 1]:

* **Desenvolvimento de Core Services:** Implementação do esquema de banco de dados relacional (entidades de Usuário, Sessão e Fatura) e serviços de persistência[cite: 1].
* **Mecanismo de Simulação e Telemetria:** Construção de simuladores de payloads do carregador para validação do motor de cálculo de rateio individualizado em tempo real[cite: 1].
* **Pipelines Iniciais de IA:** Estruturação de scripts em Python utilizando a biblioteca Scikit-learn para os algoritmos de detecção de anomalias e modelos iniciais de regressão de carga[cite: 1].
* **Arquitetura da Stack Tecnológica Recomendada:** O ecossistema utilizará Python para os microsserviços de IA e processamento de dados, combinando frameworks rápidos como FastAPI ou Flask no desenvolvimento de APIs de integração, base de dados SQL estável, e dashboards Web interativos para as áreas de usuário e gestão[cite: 1].

> **Anexo Arquitetural de Referência:** Diagrama de Fluxo e Engenharia de Sistemas[cite: 1]
