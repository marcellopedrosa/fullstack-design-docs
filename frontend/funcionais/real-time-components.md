# Componentes em Tempo Real

Este documento descreve a arquitetura, funcionamento e implementação do sistema de componentes em tempo real do CSC, permitindo que componentes de UI como gráficos, grids e visualizações se atualizem automaticamente em intervalos configuráveis.

## Visão Geral

O sistema de tempo real foi projetado seguindo os princípios de:

- **Reutilização**: Arquitetura modular que permite adicionar funcionalidade de tempo real a qualquer componente
- **Configurabilidade**: Intervalos de atualização personalizáveis por componente
- **Eficiência**: Gerenciamento centralizado de atualizações
- **Experiência do Usuário**: Indicadores visuais e controles intuitivos
- **Resiliência**: Tratamento de falhas e tentativas de reconexão

## Arquitetura

O sistema de tempo real é composto por cinco elementos principais:

1. **Hook `useRealTime`**: Hook individual para componentes independentes
2. **Contexto `RealTimeProvider`**: Gerenciador global de componentes com tempo real
3. **HOC `withRealTime`**: Higher-Order Component para adicionar funcionalidade de tempo real
4. **Componente `StatusIndicator`**: Feedback visual sobre o estado da atualização
5. **Componente `RealTimeControls`**: Interface para gerenciar atualizações globalmente

### Diagrama de Componentes

```
┌─────────────────────────────────────────────────────────┐
│ RealTimeProvider (Contexto Global)                      │
│                                                         │
│  ┌─────────────────┐   ┌─────────────────┐              │
│  │ Componente com  │   │ Componente com  │              │
│  │ tempo real      │   │ tempo real      │  ...         │
│  └─────────────────┘   └─────────────────┘              │
│                                                         │
│  ┌─────────────────────────────────────────────┐        │
│  │ RealTimeControls (Controles Globais)        │        │
│  └─────────────────────────────────────────────┘        │
└─────────────────────────────────────────────────────────┘
```

## Componentes do Sistema

### 1. Hook `useRealTime`

O hook `useRealTime` é a implementação mais básica da funcionalidade de tempo real. Pode ser usado diretamente em componentes individuais quando não é necessário um gerenciamento global.

**Localização**: `hooks/useRealTime.ts`

**Características**:
- Gerencia o ciclo de vida de atualizações periódicas
- Controla estados de carregamento, erro e dados
- Oferece métodos para iniciar, parar e forçar atualizações
- Implementa lógica de retry em caso de falha

**Exemplo de uso**:

```tsx
function MyComponent() {
  const { 
    data, 
    isLoading, 
    error, 
    start, 
    stop, 
    refresh 
  } = useRealTime({
    interval: 30000, // 30 segundos
    fetchFunction: async () => {
      const response = await fetch('/api/data');
      return response.json();
    },
    autoStart: true
  });

  // Renderização do componente usando os dados e estado
}
```

### 2. Contexto `RealTimeProvider`

O `RealTimeProvider` é um contexto React que gerencia centralmente todos os componentes com atualização em tempo real, permitindo controle global e otimização de recursos.

**Localização**: `providers/RealTimeProvider.tsx`

**Características**:
- Registra e controla múltiplos componentes com tempo real
- Gerencia eficientemente os intervalos de atualização
- Permite pausar/resumir todas as atualizações de uma vez
- Oferece métodos para atualizar componentes específicos ou todos

**Implementação**:

Deve ser adicionado ao layout principal da aplicação:

```tsx
// Em app/layout.tsx ou similar
import { RealTimeProvider } from '@/providers/RealTimeProvider';

export default function Layout({ children }) {
  return (
    <html lang="pt-BR">
      <body>
        <RealTimeProvider>
          {children}
        </RealTimeProvider>
      </body>
    </html>
  );
}
```

### 3. HOC `withRealTime`

O `withRealTime` é um Higher-Order Component que facilita a adição de funcionalidade de tempo real a qualquer componente existente.

**Localização**: `components/real-time/with-real-time.tsx`

**Características**:
- Integra-se ao contexto RealTimeProvider
- Injeta props relacionadas a tempo real no componente envolvido
- Gerencia o ciclo de vida do componente em relação às atualizações
- Fornece API consistente para todos os componentes de tempo real

**API injetada**:

O HOC adiciona as seguintes props ao componente envolvido:

| Prop | Tipo | Descrição |
|------|------|-----------|
| realTimeData | any | Dados atualizados em tempo real |
| isLoading | boolean | Estado de carregamento atual |
| lastUpdated | Date \| null | Timestamp da última atualização |
| refresh | () => Promise<void> | Força uma atualização imediata |
| startRealTime | () => void | Inicia atualizações automáticas |
| stopRealTime | () => void | Para atualizações automáticas |
| realTimeEnabled | boolean | Estado atual (ativado/desativado) |
| setUpdateInterval | (interval: number) => void | Configura o intervalo |
| interval | number | Intervalo atual em milissegundos |

### 4. Componente `StatusIndicator`

O `StatusIndicator` é um componente visual que exibe o estado atual das atualizações em tempo real.

**Localização**: `components/real-time/status-indicator.tsx`

**Características**:
- Indicador visual colorido para diferentes estados
- Tooltip com informações detalhadas
- Formatação amigável do tempo desde a última atualização
- Customizável em tamanho e aparência

### 5. Componente `RealTimeControls`

O `RealTimeControls` oferece uma interface para controlar globalmente as atualizações em tempo real.

**Localização**: `components/real-time/real-time-controls.tsx`

**Características**:
- Interface para pausar/retomar todas as atualizações
- Botão para atualizar todos os componentes manualmente
- Exibe o estado atual do sistema
- Customizável em aparência e funcionalidades exibidas

## Como Utilizar em Novos Componentes

### Abordagem Recomendada: Usando o HOC `withRealTime`

Esta é a maneira mais simples e recomendada de adicionar funcionalidade de tempo real a um componente existente:

#### Passo 1: Criar o componente base

Crie o componente que exibirá os dados, recebendo as props de tempo real:

```tsx
// Componente base que receberá as props do withRealTime
function BaseMyComponent({
  // Props específicas do seu componente
  title,
  endpoint,
  // Props injetadas pelo HOC withRealTime
  realTimeData,
  isLoading,
  lastUpdated,
  refresh,
  startRealTime,
  stopRealTime,
  realTimeEnabled,
  setUpdateInterval,
  interval
}: MyComponentProps & WithRealTimeProps) {
  
  // Usar dados em tempo real
  const data = realTimeData || [];
  
  return (
    <div>
      <header>
        <h2>{title}</h2>
        <StatusIndicator 
          lastUpdated={lastUpdated}
          enabled={realTimeEnabled}
          isLoading={isLoading}
        />
        
        {/* Controles de tempo real */}
        <div>
          <button onClick={refresh}>Atualizar agora</button>
          <button onClick={realTimeEnabled ? stopRealTime : startRealTime}>
            {realTimeEnabled ? 'Parar' : 'Iniciar'} atualizações
          </button>
        </div>
      </header>
      
      {/* Conteúdo principal */}
      <div>
        {/* Renderização dos dados */}
        {isLoading ? <Spinner /> : /* Renderizar seus dados */}
      </div>
    </div>
  );
}
```

#### Passo 2: Aplicar o HOC withRealTime

Envolva o componente com o HOC `withRealTime`:

```tsx
// Exportar componente com funcionalidade de tempo real
export const MyRealTimeComponent = withRealTime<MyComponentProps>(
  BaseMyComponent,
  {
    interval: 30000, // 30 segundos por padrão
    autoStart: true,
    displayName: 'MyRealTimeComponent',
    fetchData: async (props) => {
      // Implementar lógica de busca usando props.endpoint ou outro método
      const response = await fetch(props.endpoint);
      return response.json();
    }
  }
);
```

#### Passo 3: Usar o componente

Use o componente em sua aplicação:

```tsx
function Dashboard() {
  return (
    <div>
      <MyRealTimeComponent 
        title="Dados em Tempo Real"
        endpoint="/api/some-data"
      />
    </div>
  );
}
```

### Abordagem Alternativa: Usando o Hook useRealTime

Para casos onde você precisa de mais controle ou uma implementação mais específica:

```tsx
function CustomRealTimeComponent({ endpoint }) {
  const { 
    data, 
    isLoading, 
    error, 
    refresh, 
    start, 
    stop, 
    status, 
    lastUpdated 
  } = useRealTime({
    interval: 30000,
    fetchFunction: async () => {
      const response = await fetch(endpoint);
      return response.json();
    }
  });
  
  // Implementação personalizada do componente
}
```

## Exemplos

### AlertsChart

// ... existing code ...

### WeeklyAlertsChart

O componente `WeeklyAlertsChart` é um exemplo de aplicação do sistema de real-time para visualização de dados de alertas semanais.

#### Implementação Base

```tsx
// Componente base sem real-time
import React from 'react';
import { Bar } from 'react-chartjs-2';
import { Chart as ChartJS, CategoryScale, LinearScale, BarElement, Title, Tooltip, Legend } from 'chart.js';

// Registrando os componentes necessários do Chart.js
ChartJS.register(CategoryScale, LinearScale, BarElement, Title, Tooltip, Legend);

// Tipagem dos dados
export interface WeeklyAlertsChartData {
  labels: string[];
  values: number[];
  previousValues?: number[];
}

interface WeeklyAlertsChartProps {
  data: WeeklyAlertsChartData;
  loading?: boolean;
  error?: Error | null;
}

const WeeklyAlertsChart: React.FC<WeeklyAlertsChartProps> = ({ 
  data, 
  loading = false,
  error = null 
}) => {
  const chartData = {
    labels: data.labels,
    datasets: [
      {
        label: 'Alertas da Semana Atual',
        data: data.values,
        backgroundColor: 'rgba(53, 162, 235, 0.8)',
      },
      ...(data.previousValues ? [
        {
          label: 'Alertas da Semana Anterior',
          data: data.previousValues,
          backgroundColor: 'rgba(53, 162, 235, 0.3)',
        }
      ] : []),
    ],
  };

  const options = {
    responsive: true,
    plugins: {
      legend: {
        position: 'top' as const,
      },
      title: {
        display: true,
        text: 'Alertas Semanais',
      },
    },
  };

  if (loading) {
    return <div className="flex justify-center items-center h-64">Carregando...</div>;
  }

  if (error) {
    return <div className="flex justify-center items-center h-64 text-red-500">Erro ao carregar dados: {error.message}</div>;
  }

  return (
    <div className="p-4 border rounded-lg bg-white shadow-sm">
      <Bar data={chartData} options={options} />
    </div>
  );
};

export default WeeklyAlertsChart;
```

#### Aplicando o HOC withRealTime

```tsx
// Aplicando o HOC withRealTime
import { withRealTime } from '@/components/RealTime/withRealTime';
import { WeeklyAlertsChartData } from './WeeklyAlertsChart';

// Definição das props específicas do real-time
export interface WeeklyAlertsChartRealTimeProps {
  clientId: string;
  weekStart: string;
  weekEnd: string;
}

// Aplicando o HOC
const WeeklyAlertsChartWithRealTime = withRealTime<
  WeeklyAlertsChartData,
  WeeklyAlertsChartRealTimeProps
>({
  baseComponent: WeeklyAlertsChart,
  initialState: {
    labels: [],
    values: [],
  },
  fetchFunction: async (props) => {
    // Fazendo a chamada inicial
    const response = await fetch(`/api/alerts/weekly?clientId=${props.clientId}&weekStart=${props.weekStart}&weekEnd=${props.weekEnd}`);
    if (!response.ok) {
      throw new Error('Falha ao buscar dados de alertas');
    }
    return await response.json();
  },
  socketConfig: {
    event: 'weekly-alerts-update',
    channel: (props) => `client:${props.clientId}:alerts:weekly`,
    dataTransformer: (socketData) => {
      // Transformando os dados recebidos via socket para o formato adequado
      return {
        labels: socketData.days,
        values: socketData.alertCounts,
        previousValues: socketData.previousAlertCounts
      };
    }
  },
  updateInterval: 60000, // Atualiza a cada minuto (opcional)
  allowPause: true, // Permite pausar as atualizações
});

export default WeeklyAlertsChartWithRealTime;
```

#### Uso do Componente

```tsx
// Página ou componente pai
import WeeklyAlertsChartWithRealTime from '@/components/Charts/WeeklyAlertsChartWithRealTime';
import { format, startOfWeek, endOfWeek } from 'date-fns';

const Dashboard = () => {
  const today = new Date();
  const weekStart = format(startOfWeek(today), 'yyyy-MM-dd');
  const weekEnd = format(endOfWeek(today), 'yyyy-MM-dd');
  
  return (
    <div className="p-6">
      <h2 className="text-xl font-semibold mb-4">Visão Semanal de Alertas</h2>
      
      <WeeklyAlertsChartWithRealTime
        clientId="client123"
        weekStart={weekStart}
        weekEnd={weekEnd}
      />
      
      {/* Os controles de real-time serão incluídos automaticamente pelo HOC */}
    </div>
  );
};

export default Dashboard;
```

### Outros exemplos

// ... existing code ...

## Considerações de Performance

Para garantir o melhor desempenho ao usar componentes de tempo real:

1. **Otimize o número de componentes**: Use com moderação, especialmente em intervalos curtos
2. **Ajuste o intervalo adequadamente**: Use intervalos maiores quando possível
3. **Use a pausa global**: Implemente a possibilidade de pausar atualizações quando não visíveis
4. **Considere o impacto no servidor**: Balanceie a frequência das atualizações considerando a carga no servidor
5. **Implemente cache**: Para dados que mudam pouco, considere políticas de cache

## Boas Práticas

1. **Feedback Visual**: Sempre use o `StatusIndicator` para que o usuário saiba o estado das atualizações
2. **Controle do Usuário**: Permita que o usuário controle as atualizações
3. **Tratamento de Erros**: Implemente tratamento adequado de falhas
4. **Cleanup**: Garanta que os intervalos sejam limpos ao desmontar componentes
5. **Estado de Carregamento**: Indique visualmente quando uma atualização está em andamento
6. **Configurabilidade**: Permita configurar o intervalo para diferentes necessidades

## Conclusão

O sistema de componentes em tempo real oferece uma solução flexível e reutilizável para adicionar funcionalidade de atualização automática a qualquer componente da interface. Seguindo os padrões e diretrizes descritos nesta documentação, você pode implementar facilmente novos componentes que se beneficiam de atualizações regulares de dados. 