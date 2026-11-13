import React, { useState, useEffect, useMemo } from 'react';
import {
  TrendingUp,
  TrendingDown,
  ShieldAlert,
  Target,
  Activity,
  DollarSign,
  Plus,
  Edit2,
  Trash2,
  BarChart3,
  Calendar,
  RefreshCw,
  Award,
  Sliders,
  X,
  Check,
  ArrowUpRight,
  ArrowDownRight,
  Lock,
  BookOpen,
  Image as ImageIcon,
  Camera,
  Eye,
  Crosshair,
  Layers
} from 'lucide-react';
import {
  ResponsiveContainer,
  ComposedChart,
  Line,
  Bar,
  XAxis,
  YAxis,
  CartesianGrid,
  Tooltip,
  ReferenceLine,
  Cell,
  Area
} from 'recharts';

// --- DADOS INICIAIS DE EXEMPLO (Para visualização imediata dos gráficos) ---
const getTodayStr = () => new Date().toISOString().split('T')[0];

const getPastDateStr = (daysAgo) => {
  const d = new Date();
  d.setDate(d.getDate() - daysAgo);
  return d.toISOString().split('T')[0];
};

const INITIAL_SETTINGS = {
  initialCapital: 5000,
  monthlyTargetProfit: 2500, // Quanto quer buscar no mês (R$)
  dailyStopLoss: 250,        // Stop loss diário em R$
  maxDailyTrades: 5,         // Quantidade máxima de entradas diárias
  maxConsecutiveLosses: 2,   // Limite de stops consecutivos no dia
  maxDailyLosses: 3,         // Quantidade máxima de stops no dia
  traderName: 'Trader Pro'
};

const INITIAL_TRADES = [
  { id: '1', date: getPastDateStr(18), asset: 'WIN', type: 'Compra', entryValue: '128.450 pts (5 cts)', strategy: 'Rompimento de Abertura', result: 320, notes: 'Fluxo comprador forte na abertura', screenshot: null },
  { id: '2', date: getPastDateStr(18), asset: 'WDO', type: 'Venda', entryValue: '5.482,5 pts (2 cts)', strategy: 'Pullback VWAP', result: -140, notes: 'Stop técnico na VWAP', screenshot: null },
  { id: '3', date: getPastDateStr(15), asset: 'WIN', type: 'Compra', entryValue: '129.100 pts (5 cts)', strategy: 'Pullback Média 20', result: 410, notes: 'Tendência limpa', screenshot: null },
  { id: '4', date: getPastDateStr(12), asset: 'WIN', type: 'Venda', entryValue: '130.200 pts (4 cts)', strategy: 'Exaustão de Topo', result: 280, notes: 'Divergência no IFR', screenshot: null },
  { id: '5', date: getPastDateStr(10), asset: 'WDO', type: 'Compra', entryValue: '5.510,0 pts (2 cts)', strategy: 'Rompimento Range', result: -220, notes: 'Volatilidade Payroll', screenshot: null },
  { id: '6', date: getPastDateStr(8), asset: 'WIN', type: 'Compra', entryValue: '127.800 pts (6 cts)', strategy: 'Tendência Primária', result: 490, notes: 'Seguindo fluxo institucional', screenshot: null },
  { id: '7', date: getPastDateStr(5), asset: 'BTC', type: 'Compra', entryValue: '$64.250 (0.05 BTC)', strategy: 'Suporte 4H + Volume', result: 360, notes: 'Confirmação de fundo duplo', screenshot: null },
  { id: '8', date: getPastDateStr(3), asset: 'WIN', type: 'Venda', entryValue: '128.900 pts (3 cts)', strategy: 'Contra-tendência', result: -110, notes: 'Entrada precipitada antes do fechamento do candle', screenshot: null },
  { id: '9', date: getPastDateStr(3), asset: 'WIN', type: 'Compra', entryValue: '129.050 pts (5 cts)', strategy: 'Pullback VWAP', result: 340, notes: 'Recuperação disciplinada no plano', screenshot: null },
  { id: '10', date: getTodayStr(), asset: 'WIN', type: 'Compra', entryValue: '131.120 pts (5 cts)', strategy: 'Rompimento Pivô', result: 260, notes: 'Primeira entrada do dia cravada no alvo', screenshot: null },
];

const INITIAL_JOURNAL = {
  [getTodayStr()]: {
    mood: 'Focado',
    content: 'Mercado abriu com boa liquidez. Respeitei o plano operacional, aguardei a confirmação do pivô e bati a meta parcial logo na primeira operação sem ansiedade.'
  }
};

// --- UTILITÁRIOS DE FORMATAÇÃO E IMAGEM ---
const compressImage = (file, maxWidth = 1000, quality = 0.72) => {
  return new Promise((resolve, reject) => {
    const reader = new FileReader();
    reader.readAsDataURL(file);
    reader.onload = (event) => {
      const img = new Image();
      img.src = event.target.result;
      img.onload = () => {
        const canvas = document.createElement('canvas');
        let width = img.width;
        let height = img.height;
        if (width > maxWidth) {
          height = Math.round((height * maxWidth) / width);
          width = maxWidth;
        }
        canvas.width = width;
        canvas.height = height;
        const ctx = canvas.getContext('2d');
        ctx.drawImage(img, 0, 0, width, height);
        resolve(canvas.toDataURL('image/jpeg', quality));
      };
      img.onerror = (err) => reject(err);
    };
    reader.onerror = (err) => reject(err);
  });
};

const formatCurrency = (value) => {
  const num = Number(value) || 0;
  return num.toLocaleString('pt-BR', {
    style: 'currency',
    currency: 'BRL',
    minimumFractionDigits: 2
  });
};

const formatDateBR = (dateStr) => {
  if (!dateStr) return '';
  const [year, month, day] = dateStr.split('-');
  return `${day}/${month}/${year}`;
};

const getWeekLabel = (dateStr) => {
  const date = new Date(dateStr + 'T12:00:00');
  const firstDayOfYear = new Date(date.getFullYear(), 0, 1);
  const pastDaysOfYear = (date - firstDayOfYear) / 86400000;
  const weekNum = Math.ceil((pastDaysOfYear + firstDayOfYear.getDay() + 1) / 7);
  return `Sem ${weekNum}/${String(date.getFullYear()).slice(2)}`;
};

const getMonthLabel = (dateStr) => {
  const months = ['Jan', 'Fev', 'Mar', 'Abr', 'Mai', 'Jun', 'Jul', 'Ago', 'Set', 'Out', 'Nov', 'Dez'];
  const [year, month] = dateStr.split('-');
  return `${months[parseInt(month, 10) - 1]}/${year.slice(2)}`;
};

// --- COMPONENTE PRINCIPAL ---
export default function App() {
  // Estados com persistência no localStorage
  const [settings, setSettings] = useState(() => {
    try {
      const saved = localStorage.getItem('dt_risk_settings_v1');
      return saved ? JSON.parse(saved) : INITIAL_SETTINGS;
    } catch {
      return INITIAL_SETTINGS;
    }
  });

  const [trades, setTrades] = useState(() => {
    try {
      const saved = localStorage.getItem('dt_risk_trades_v1');
      return saved ? JSON.parse(saved) : INITIAL_TRADES;
    } catch {
      return INITIAL_TRADES;
    }
  });

  // Estados de interface
  const [chartPeriod, setChartPeriod] = useState('daily'); // 'daily' | 'weekly' | 'monthly'
  const [showResetConfirm, setShowResetConfirm] = useState(false);
  const [editingTradeId, setEditingTradeId] = useState(null);
  const [filterType, setFilterType] = useState('ALL'); // 'ALL' | 'GAIN' | 'LOSS'
  const [previewTrade, setPreviewTrade] = useState(null); // Modal para visualizar print e detalhes

  // Estado do Calendário / Visualização Mensal
  const [calendarYearMonth, setCalendarYearMonth] = useState(() => {
    const now = new Date();
    const y = now.getFullYear();
    const m = String(now.getMonth() + 1).padStart(2, '0');
    return `${y}-${m}`;
  });

  // Estado do Diário de Trader Diário
  const [journalEntries, setJournalEntries] = useState(() => {
    try {
      const saved = localStorage.getItem('dt_risk_journal_v1');
      return saved ? JSON.parse(saved) : INITIAL_JOURNAL;
    } catch {
      return INITIAL_JOURNAL;
    }
  });
  const [journalDate, setJournalDate] = useState(getTodayStr());
  const [journalSavedFeedback, setJournalSavedFeedback] = useState(false);

  // Formulário de Nova Operação / Edição
  const [formData, setFormData] = useState({
    date: getTodayStr(),
    asset: 'WIN',
    type: 'Compra',
    entryValue: '',
    strategy: '',
    outcome: 'GAIN', // 'GAIN' (WIN) | 'LOSS' (STOP)
    amount: '',
    notes: '',
    screenshot: null
  });

  // Sincronização com localStorage
  useEffect(() => {
    try {
      localStorage.setItem('dt_risk_settings_v1', JSON.stringify(settings));
    } catch (e) {
      console.error('Erro ao salvar configurações:', e);
    }
  }, [settings]);

  useEffect(() => {
    try {
      localStorage.setItem('dt_risk_trades_v1', JSON.stringify(trades));
    } catch (e) {
      console.error('Erro ao salvar operações:', e);
    }
  }, [trades]);

  useEffect(() => {
    try {
      localStorage.setItem('dt_risk_journal_v1', JSON.stringify(journalEntries));
    } catch (e) {
      console.error('Erro ao salvar diário:', e);
    }
  }, [journalEntries]);

  // --- CÁLCULOS DE MÉTRICAS E RISCO ---
  const metrics = useMemo(() => {
    const sortedTrades = [...trades].sort((a, b) => a.date.localeCompare(b.date));
    const totalProfit = sortedTrades.reduce((acc, t) => acc + Number(t.result), 0);
    const currentCapital = Number(settings.initialCapital) + totalProfit;
    const targetCapital = Number(settings.initialCapital) + Number(settings.monthlyTargetProfit);

    // Progresso da meta mensal (baseado no lucro acumulado vs lucro projetado)
    const monthlyTargetProfit = Number(settings.monthlyTargetProfit) || 1;
    const goalProgressRaw = (totalProfit / monthlyTargetProfit) * 100;
    const goalProgressClamped = Math.max(0, Math.min(100, goalProgressRaw));
    const projectedReturnPct = settings.initialCapital > 0
      ? ((Number(settings.monthlyTargetProfit) / Number(settings.initialCapital)) * 100).toFixed(1)
      : '0.0';

    // Assertividade (Win Rate)
    const wins = sortedTrades.filter((t) => Number(t.result) > 0);
    const losses = sortedTrades.filter((t) => Number(t.result) < 0);
    const totalTradesCount = sortedTrades.length;
    const winRate = totalTradesCount > 0 ? ((wins.length / totalTradesCount) * 100).toFixed(1) : '0.0';

    // Payoff e Fator de Lucro
    const grossProfit = wins.reduce((acc, t) => acc + Number(t.result), 0);
    const grossLoss = Math.abs(losses.reduce((acc, t) => acc + Number(t.result), 0));
    const avgWin = wins.length > 0 ? grossProfit / wins.length : 0;
    const avgLoss = losses.length > 0 ? grossLoss / losses.length : 0;
    const payoff = avgLoss > 0 ? (avgWin / avgLoss).toFixed(2) : wins.length > 0 ? '∞' : '0.00';
    const profitFactor = grossLoss > 0 ? (grossProfit / grossLoss).toFixed(2) : grossProfit > 0 ? '∞' : '0.00';

    // Risco por Entrada e Stop Loss %
    const maxTrades = Math.max(1, Number(settings.maxDailyTrades) || 1);
    const dailyStop = Number(settings.dailyStopLoss) || 0;
    const riskPerTrade = dailyStop / maxTrades;
    const stopLossPct = settings.initialCapital > 0
      ? ((dailyStop / Number(settings.initialCapital)) * 100).toFixed(2)
      : '0.00';

    // Status do Dia Atual (Hoje)
    const todayStr = getTodayStr();
    const todayTrades = sortedTrades
      .filter((t) => t.date === todayStr)
      .sort((a, b) => Number(a.id) - Number(b.id));
    const todayResult = todayTrades.reduce((acc, t) => acc + Number(t.result), 0);
    const todayCount = todayTrades.length;

    // Contagem de Stops no dia e Stops Consecutivos atuais (hoje)
    const todayLossesCount = todayTrades.filter((t) => Number(t.result) < 0).length;
    let currentConsecutiveLosses = 0;
    for (let i = todayTrades.length - 1; i >= 0; i--) {
      if (Number(todayTrades[i].result) < 0) {
        currentConsecutiveLosses += 1;
      } else {
        break;
      }
    }

    const maxConsecLosses = Math.max(1, Number(settings.maxConsecutiveLosses) || 2);
    const maxDayLosses = Math.max(1, Number(settings.maxDailyLosses) || 3);

    const isDailyStopHit = dailyStop > 0 && todayResult <= -dailyStop;
    const isConsecutiveStopHit = currentConsecutiveLosses >= maxConsecLosses;
    const isDailyLossCountHit = todayLossesCount >= maxDayLosses;
    const isMaxTradesHit = todayCount >= maxTrades;
    const remainingDailyRisk = Math.max(0, dailyStop + todayResult);

    return {
      totalProfit,
      currentCapital,
      targetCapital,
      goalProgressRaw: goalProgressRaw.toFixed(1),
      goalProgressClamped,
      projectedReturnPct,
      winRate,
      winsCount: wins.length,
      lossesCount: losses.length,
      totalTradesCount,
      payoff,
      profitFactor,
      riskPerTrade,
      stopLossPct,
      todayResult,
      todayCount,
      todayLossesCount,
      currentConsecutiveLosses,
      isDailyStopHit,
      isConsecutiveStopHit,
      isDailyLossCountHit,
      isMaxTradesHit,
      remainingDailyRisk
    };
  }, [trades, settings]);

  // --- CÁLCULO DO RESULTADO MENSAL ATUAL ---
  const currentMonthResult = useMemo(() => {
    const [year, month] = calendarYearMonth.split('-');
    return trades
      .filter((t) => {
        if (!t.date) return false;
        const [tYear, tMonth] = t.date.split('-');
        return tYear === year && tMonth === month;
      })
      .reduce((acc, t) => acc + Number(t.result), 0);
  }, [trades, calendarYearMonth]);

  // --- PREPARAÇÃO DE DADOS PARA O CALENDÁRIO MENSAL (DIA A DIA) ---
  const calendarDaysData = useMemo(() => {
    const [yearStr, monthStr] = calendarYearMonth.split('-');
    const year = parseInt(yearStr, 10);
    const month = parseInt(monthStr, 10) - 1; // 0-indexed

    const firstDayIndex = new Date(year, month, 1).getDay(); // 0 (Dom) a 6 (Sáb)
    const totalDaysInMonth = new Date(year, month + 1, 0).getDate();

    // Mapear resultados por dia do mês selecionado
    const dailyMap = {};
    trades.forEach((t) => {
      if (!t.date) return;
      const [tY, tM, tD] = t.date.split('-');
      if (parseInt(tY, 10) === year && parseInt(tM, 10) === (month + 1)) {
        const dayNum = parseInt(tD, 10);
        if (!dailyMap[dayNum]) {
          dailyMap[dayNum] = { pnl: 0, count: 0, wins: 0, losses: 0 };
        }
        const res = Number(t.result);
        dailyMap[dayNum].pnl += res;
        dailyMap[dayNum].count += 1;
        if (res > 0) dailyMap[dayNum].wins += 1;
        if (res < 0) dailyMap[dayNum].losses += 1;
      }
    });

    const daysArray = [];
    // Espaços vazios antes do dia 1
    for (let i = 0; i < firstDayIndex; i++) {
      daysArray.push({ day: null });
    }
    // Dias do mês
    for (let d = 1; d <= totalDaysInMonth; d++) {
      const dateStr = `${yearStr}-${monthStr}-${String(d).padStart(2, '0')}`;
      daysArray.push({
        day: d,
        dateStr,
        data: dailyMap[d] || null
      });
    }
    return daysArray;
  }, [trades, calendarYearMonth]);

  // --- PREPARAÇÃO DE DADOS PARA O GRÁFICO (DIÁRIO, SEMANAL E MENSAL) ---
  const chartData = useMemo(() => {
    const sorted = [...trades].sort((a, b) => a.date.localeCompare(b.date));
    const initialCap = Number(settings.initialCapital) || 0;
    const targetCap = initialCap + (Number(settings.monthlyTargetProfit) || 0);

    if (sorted.length === 0) {
      return [
        {
          label: 'Início',
          resultadoPeriodo: 0,
          capitalAcumulado: initialCap,
          metaCapital: targetCap
        }
      ];
    }

    if (chartPeriod === 'daily') {
      const map = new Map();
      sorted.forEach((t) => {
        const key = t.date;
        const prev = map.get(key) || { date: key, resultado: 0, count: 0 };
        prev.resultado += Number(t.result);
        prev.count += 1;
        map.set(key, prev);
      });

      let runningCapital = initialCap;
      const result = [
        {
          label: 'Início',
          fullDate: 'Capital Inicial',
          resultadoPeriodo: 0,
          capitalAcumulado: initialCap,
          metaCapital: targetCap
        }
      ];

      Array.from(map.values()).forEach((item) => {
        runningCapital += item.resultado;
        result.push({
          label: formatDateBR(item.date).slice(0, 5),
          fullDate: formatDateBR(item.date),
          resultadoPeriodo: item.resultado,
          capitalAcumulado: runningCapital,
          metaCapital: targetCap,
          trades: item.count
        });
      });

      return result;
    }

    if (chartPeriod === 'weekly') {
      const map = new Map();
      sorted.forEach((t) => {
        const key = getWeekLabel(t.date);
        const prev = map.get(key) || { label: key, resultado: 0, count: 0 };
        prev.resultado += Number(t.result);
        prev.count += 1;
        map.set(key, prev);
      });

      let runningCapital = initialCap;
      return Array.from(map.values()).map((item) => {
        runningCapital += item.resultado;
        return {
          label: item.label,
          fullDate: item.label,
          resultadoPeriodo: item.resultado,
          capitalAcumulado: runningCapital,
          metaCapital: targetCap,
          trades: item.count
        };
      });
    }

    const map = new Map();
    sorted.forEach((t) => {
      const key = getMonthLabel(t.date);
      const prev = map.get(key) || { label: key, resultado: 0, count: 0 };
      prev.resultado += Number(t.result);
      prev.count += 1;
      map.set(key, prev);
    });

    let runningCapital = initialCap;
    return Array.from(map.values()).map((item) => {
      runningCapital += item.resultado;
      return {
        label: item.label,
        fullDate: item.label,
        resultadoPeriodo: item.resultado,
        capitalAcumulado: runningCapital,
        metaCapital: targetCap,
        trades: item.count
      };
    });
  }, [trades, settings, chartPeriod]);

  // --- MANIPULADORES DE AÇÕES ---
  const handleSettingChange = (field, value) => {
    const numValue = parseFloat(value);
    setSettings((prev) => ({
      ...prev,
      [field]: isNaN(numValue) ? 0 : Math.max(0, numValue)
    }));
  };

  const handleSaveTrade = (e) => {
    e.preventDefault();
    const rawAmount = Math.abs(parseFloat(String(formData.amount).replace(',', '.')));
    if (isNaN(rawAmount) || rawAmount <= 0) return;

    const finalResult = formData.outcome === 'GAIN' ? rawAmount : -rawAmount;

    if (editingTradeId) {
      setTrades((prev) =>
        prev.map((t) =>
          t.id === editingTradeId
            ? {
                ...t,
                date: formData.date,
                asset: formData.asset.toUpperCase() || 'ATIVO',
                type: formData.type,
                entryValue: formData.entryValue,
                strategy: formData.strategy,
                result: finalResult,
                notes: formData.notes,
                screenshot: formData.screenshot
              }
            : t
        )
      );
      setEditingTradeId(null);
    } else {
      const newTrade = {
        id: String(Date.now()),
        date: formData.date || getTodayStr(),
        asset: formData.asset.toUpperCase() || 'WIN',
        type: formData.type,
        entryValue: formData.entryValue,
        strategy: formData.strategy,
        result: finalResult,
        notes: formData.notes,
        screenshot: formData.screenshot
      };
      setTrades((prev) => [newTrade, ...prev]);
    }

    setFormData((prev) => ({
      ...prev,
      entryValue: '',
      strategy: '',
      amount: '',
      notes: '',
      screenshot: null
    }));
  };

  const handleStartEdit = (trade) => {
    setEditingTradeId(trade.id);
    setFormData({
      date: trade.date,
      asset: trade.asset,
      type: trade.type,
      entryValue: trade.entryValue || '',
      strategy: trade.strategy || '',
      outcome: Number(trade.result) >= 0 ? 'GAIN' : 'LOSS',
      amount: String(Math.abs(Number(trade.result))),
      notes: trade.notes || '',
      screenshot: trade.screenshot || null
    });
    const formEl = document.getElementById('trade-form-section');
    if (formEl) formEl.scrollIntoView({ behavior: 'smooth', block: 'center' });
  };

  const handleCancelEdit = () => {
    setEditingTradeId(null);
    setFormData({
      date: getTodayStr(),
      asset: 'WIN',
      type: 'Compra',
      entryValue: '',
      strategy: '',
      outcome: 'GAIN',
      amount: '',
      notes: '',
      screenshot: null
    });
  };

  const handleScreenshotUpload = async (e) => {
    const file = e.target.files?.[0];
    if (!file) return;
    try {
      const compressedDataUrl = await compressImage(file);
      setFormData((prev) => ({ ...prev, screenshot: compressedDataUrl }));
    } catch (err) {
      console.error('Erro ao processar imagem:', err);
    }
  };

  const handleJournalUpdate = (field, value) => {
    setJournalEntries((prev) => ({
      ...prev,
      [journalDate]: {
        mood: prev[journalDate]?.mood || 'Focado',
        content: prev[journalDate]?.content || '',
        [field]: value
      }
    }));
    setJournalSavedFeedback(true);
    setTimeout(() => setJournalSavedFeedback(false), 1500);
  };

  const handleDeleteTrade = (id) => {
    if (editingTradeId === id) {
      handleCancelEdit();
    }
    setTrades((prev) => prev.filter((t) => t.id !== id));
  };

  const handleResetAllData = () => {
    setTrades([]);
    setJournalEntries({});
    setEditingTradeId(null);
    setShowResetConfirm(false);
  };

  const handleRestoreDemoData = () => {
    setSettings(INITIAL_SETTINGS);
    setTrades(INITIAL_TRADES);
    setJournalEntries(INITIAL_JOURNAL);
    setEditingTradeId(null);
    setShowResetConfirm(false);
  };

  const currentJournal = journalEntries[journalDate] || { mood: 'Focado', content: '' };

  const selectedDateStats = useMemo(() => {
    const dayTrades = trades.filter((t) => t.date === journalDate);
    const pnl = dayTrades.reduce((acc, t) => acc + Number(t.result), 0);
    const wins = dayTrades.filter((t) => Number(t.result) > 0).length;
    const losses = dayTrades.filter((t) => Number(t.result) < 0).length;
    return { count: dayTrades.length, pnl, wins, losses };
  }, [trades, journalDate]);

  const filteredTrades = useMemo(() => {
    return [...trades]
      .sort((a, b) => b.date.localeCompare(a.date) || Number(b.id) - Number(a.id))
      .filter((t) => {
        if (filterType === 'GAIN') return Number(t.result) > 0;
        if (filterType === 'LOSS') return Number(t.result) < 0;
        return true;
      });
  }, [trades, filterType]);

  const CustomTooltip = ({ active, payload }) => {
    if (active && payload && payload.length) {
      const data = payload[0].payload;
      const isPositive = data.resultadoPeriodo >= 0;
      return (
        <div className="bg-slate-900/95 border border-slate-700/80 p-3.5 rounded-xl shadow-2xl backdrop-blur-md text-xs space-y-1.5">
          <p className="font-bold text-slate-200 border-b border-slate-800 pb-1">
            {data.fullDate || data.label}
          </p>
          <div className="flex items-center justify-between gap-4">
            <span className="text-slate-400">Resultado Período:</span>
            <span className={`font-bold ${isPositive ? 'text-emerald-400' : 'text-red-400'}`}>
              {isPositive ? '+' : ''}
              {formatCurrency(data.resultadoPeriodo)}
            </span>
          </div>
          <div className="flex items-center justify-between gap-4">
            <span className="text-slate-400">Capital Acumulado:</span>
            <span className="font-semibold text-white">{formatCurrency(data.capitalAcumulado)}</span>
          </div>
          <div className="flex items-center justify-between gap-4">
            <span className="text-slate-400">Meta Projetada:</span>
            <span className="font-medium text-emerald-400/80">{formatCurrency(data.metaCapital)}</span>
          </div>
        </div>
      );
    }
    return null;
  };

  const isOverallPositive = metrics.totalProfit >= 0;

  return (
    <div className="min-h-screen bg-slate-950 text-slate-100 font-sans selection:bg-emerald-500/30 selection:text-emerald-200 pb-12">
      {/* --- BARRA SUPERIOR / HEADER --- */}
      <header className="sticky top-0 z-30 bg-slate-950/90 backdrop-blur-lg border-b border-slate-800/80">
        <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 h-16 flex items-center justify-between gap-4">
          <div className="flex items-center gap-3">
            <div className="w-10 h-10 rounded-xl bg-gradient-to-br from-emerald-500 to-emerald-700 flex items-center justify-center shadow-lg shadow-emerald-500/20">
              <Activity className="w-5 h-5 text-white" />
            </div>
            <div>
              <h1 className="text-base sm:text-lg font-bold tracking-tight text-white flex items-center gap-2">
                RiskDesk <span className="text-emerald-400">PRO</span>
              </h1>
              <p className="text-[11px] text-slate-400 hidden sm:block">
                Gestão de Risco & Alta Performance no Day Trade
              </p>
            </div>
          </div>

          <div className="flex items-center gap-2 sm:gap-3">
            {!showResetConfirm ? (
              <button
                onClick={() => setShowResetConfirm(true)}
                className="flex items-center gap-1.5 px-3 py-2 rounded-lg text-xs font-medium bg-slate-900 hover:bg-slate-800 text-slate-300 border border-slate-800 transition-colors cursor-pointer"
                title="Limpar ou restaurar dados"
              >
                <RefreshCw className="w-3.5 h-3.5 text-slate-400" />
                <span className="hidden sm:inline">Limpar / Resetar</span>
              </button>
            ) : (
              <div className="flex items-center gap-1.5 bg-slate-900 border border-red-500/40 px-2.5 py-1.5 rounded-lg">
                <span className="text-[11px] text-slate-300 mr-1 hidden md:inline">Ação:</span>
                <button
                  onClick={handleResetAllData}
                  className="px-2 py-1 rounded bg-red-500/20 hover:bg-red-500/30 text-red-400 text-[11px] font-semibold transition-colors cursor-pointer"
                >
                  Zerar Tudo
                </button>
                <button
                  onClick={handleRestoreDemoData}
                  className="px-2 py-1 rounded bg-emerald-500/20 hover:bg-emerald-500/30 text-emerald-400 text-[11px] font-semibold transition-colors cursor-pointer"
                >
                  Exemplo
                </button>
                <button
                  onClick={() => setShowResetConfirm(false)}
                  className="p-1 text-slate-400 hover:text-white cursor-pointer"
                >
                  <X className="w-3.5 h-3.5" />
                </button>
              </div>
            )}
          </div>
        </div>
      </header>

      <main className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 pt-6 space-y-6">
        {/* --- BANNER DE ALERTA DE DISCIPLINA --- */}
        {(metrics.isDailyStopHit ||
          metrics.isConsecutiveStopHit ||
          metrics.isDailyLossCountHit ||
          metrics.isMaxTradesHit) && (
          <div
            className={`rounded-2xl p-4 border flex flex-col sm:flex-row items-start sm:items-center justify-between gap-4 shadow-lg animate-pulse ${
              metrics.isDailyStopHit || metrics.isConsecutiveStopHit || metrics.isDailyLossCountHit
                ? 'bg-red-950/60 border-red-500/60 shadow-red-500/10'
                : 'bg-amber-950/50 border-amber-500/50 shadow-amber-500/10'
            }`}
          >
            <div className="flex items-start sm:items-center gap-3.5">
              <div
                className={`p-2.5 rounded-xl shrink-0 ${
                  metrics.isDailyStopHit || metrics.isConsecutiveStopHit || metrics.isDailyLossCountHit
                    ? 'bg-red-500/20 text-red-400'
                    : 'bg-amber-500/20 text-amber-400'
                }`}
              >
                <Lock className="w-6 h-6" />
              </div>
              <div>
                <h2
                  className={`font-bold text-sm sm:text-base ${
                    metrics.isDailyStopHit || metrics.isConsecutiveStopHit || metrics.isDailyLossCountHit
                      ? 'text-red-400'
                      : 'text-amber-300'
                  }`}
                >
                  {metrics.isDailyStopHit
                    ? 'ALERTA CRÍTICO: STOP LOSS FINANCEIRO DIÁRIO ATINGIDO!'
                    : metrics.isConsecutiveStopHit
                    ? `ALERTA DE DISCIPLINA: ${metrics.currentConsecutiveLosses} STOPS SEGUIDOS ATINGIDOS!`
                    : metrics.isDailyLossCountHit
                    ? `LIMITE DE STOPS NO DIA ATINGIDO (${metrics.todayLossesCount}/${settings.maxDailyLosses ?? 3})!`
                    : 'LIMITE DE ENTRADAS DIÁRIAS ATINGIDO!'}
                </h2>
                <p className="text-xs sm:text-sm text-slate-300 mt-0.5">
                  {metrics.isDailyStopHit
                    ? `Você atingiu seu limite máximo de perda diária (${formatCurrency(
                        settings.dailyStopLoss
                      )}). Feche a plataforma e proteja seu capital para amanhã.`
                    : metrics.isConsecutiveStopHit
                    ? `Você tomou ${metrics.currentConsecutiveLosses} loss consecutivos (limite configurado: ${
                        settings.maxConsecutiveLosses ?? 2
                      }). Pare agora para evitar o emocional e preservar sua banca!`
                    : metrics.isDailyLossCountHit
                    ? `Você já atingiu ${metrics.todayLossesCount} stops hoje (máximo permitido: ${
                        settings.maxDailyLosses ?? 3
                      }). Encerre seu dia operacional conforme seu plano!`
                    : `Você já realizou ${metrics.todayCount} de ${settings.maxDailyTrades} operações permitidas hoje. Evite o overtrading!`}
                </p>
              </div>
            </div>
            <span
              className={`px-3 py-1.5 rounded-lg text-xs font-bold uppercase tracking-wider shrink-0 ${
                metrics.isDailyStopHit || metrics.isConsecutiveStopHit || metrics.isDailyLossCountHit
                  ? 'bg-red-500 text-white'
                  : 'bg-amber-500 text-slate-950'
              }`}
            >
              Dia Encerrado
            </span>
          </div>
        )}

        {/* --- SEÇÃO 1: PAINEL DE PARÂMETROS DE RISCO & PROJEÇÃO --- */}
        <section className="bg-slate-900/70 border border-slate-800/90 rounded-2xl p-5 shadow-xl">
          <div className="flex flex-col sm:flex-row sm:items-center justify-between gap-2 mb-4 pb-3 border-b border-slate-800/80">
            <div className="flex items-center gap-2">
              <Sliders className="w-4 h-4 text-emerald-400" />
              <h2 className="text-sm font-bold uppercase tracking-wider text-slate-200">
                Parâmetros de Capital, Meta & Gerenciamento de Risco
              </h2>
            </div>
            <span className="text-xs text-slate-400">
              Edite os valores abaixo para recalcular toda a estratégia em tempo real
            </span>
          </div>

          <div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-4">
            <div className="bg-slate-950/80 border border-slate-800 rounded-xl p-3.5 focus-within:border-emerald-500/60 transition-colors">
              <label className="flex items-center justify-between text-xs font-medium text-slate-400 mb-1.5">
                <span>Capital Inicial (Banca)</span>
                <DollarSign className="w-3.5 h-3.5 text-emerald-400" />
              </label>
              <div className="relative flex items-center">
                <span className="text-slate-500 font-semibold text-sm mr-1.5">R$</span>
                <input
                  type="number"
                  min="0"
                  step="100"
                  value={settings.initialCapital}
                  onChange={(e) => handleSettingChange('initialCapital', e.target.value)}
                  className="w-full bg-transparent text-lg font-bold text-white focus:outline-none"
                />
              </div>
              <p className="text-[11px] text-slate-500 mt-1">Valor base para cálculo de evolução</p>
            </div>

            <div className="bg-slate-950/80 border border-slate-800 rounded-xl p-3.5 focus-within:border-emerald-500/60 transition-colors">
              <label className="flex items-center justify-between text-xs font-medium text-slate-400 mb-1.5">
                <span>Projeção de Lucro (Mês)</span>
                <Target className="w-3.5 h-3.5 text-emerald-400" />
              </label>
              <div className="relative flex items-center">
                <span className="text-emerald-500 font-semibold text-sm mr-1.5">R$</span>
                <input
                  type="number"
                  min="0"
                  step="50"
                  value={settings.monthlyTargetProfit}
                  onChange={(e) => handleSettingChange('monthlyTargetProfit', e.target.value)}
                  className="w-full bg-transparent text-lg font-bold text-emerald-400 focus:outline-none"
                />
              </div>
              <p className="text-[11px] text-emerald-400/80 mt-1">
                Alvo: <strong>+{metrics.projectedReturnPct}%</strong> ({formatCurrency(metrics.targetCapital)})
              </p>
            </div>

            <div className="bg-slate-950/80 border border-slate-800 rounded-xl p-3.5 focus-within:border-red-500/60 transition-colors">
              <label className="flex items-center justify-between text-xs font-medium text-slate-400 mb-1.5">
                <span>Stop Loss Diário Máx.</span>
                <ShieldAlert className="w-3.5 h-3.5 text-red-400" />
              </label>
              <div className="relative flex items-center">
                <span className="text-red-500 font-semibold text-sm mr-1.5">R$</span>
                <input
                  type="number"
                  min="0"
                  step="10"
                  value={settings.dailyStopLoss}
                  onChange={(e) => handleSettingChange('dailyStopLoss', e.target.value)}
                  className="w-full bg-transparent text-lg font-bold text-red-400 focus:outline-none"
                />
              </div>
              <p className="text-[11px] text-red-400/80 mt-1">
                Risco diário: <strong>{metrics.stopLossPct}%</strong> da banca inicial
              </p>
            </div>

            <div className="bg-slate-950/80 border border-slate-800 rounded-xl p-3.5 focus-within:border-emerald-500/60 transition-colors">
              <label className="flex items-center justify-between text-xs font-medium text-slate-400 mb-1.5">
                <span>Entradas Máx. / Dia</span>
                <Activity className="w-3.5 h-3.5 text-slate-400" />
              </label>
              <div className="relative flex items-center">
                <input
                  type="number"
                  min="1"
                  max="50"
                  step="1"
                  value={settings.maxDailyTrades}
                  onChange={(e) => handleSettingChange('maxDailyTrades', e.target.value)}
                  className="w-full bg-transparent text-lg font-bold text-white focus:outline-none"
                />
                <span className="text-xs text-slate-500 font-medium">
                  hoje: {metrics.todayCount}/{settings.maxDailyTrades}
                </span>
              </div>
              <p className="text-[11px] text-slate-400 mt-1">
                Risco por entrada: <strong className="text-red-400">{formatCurrency(metrics.riskPerTrade)}</strong>
              </p>
            </div>

            <div
              className={`bg-slate-950/80 border rounded-xl p-3.5 transition-colors ${
                metrics.isConsecutiveStopHit
                  ? 'border-red-500/80 bg-red-950/20'
                  : 'border-slate-800 focus-within:border-red-500/60'
              }`}
            >
              <label className="flex items-center justify-between text-xs font-medium text-slate-400 mb-1.5">
                <span>Stop Consecutivo (Seguidos)</span>
                <TrendingDown className="w-3.5 h-3.5 text-red-400" />
              </label>
              <div className="relative flex items-center">
                <input
                  type="number"
                  min="1"
                  max="20"
                  step="1"
                  value={settings.maxConsecutiveLosses ?? 2}
                  onChange={(e) => handleSettingChange('maxConsecutiveLosses', e.target.value)}
                  className="w-full bg-transparent text-lg font-bold text-red-400 focus:outline-none"
                />
                <span
                  className={`text-xs font-semibold whitespace-nowrap ${
                    metrics.isConsecutiveStopHit ? 'text-red-400' : 'text-slate-400'
                  }`}
                >
                  atual: {metrics.currentConsecutiveLosses}/{settings.maxConsecutiveLosses ?? 2}
                </span>
              </div>
              <p className="text-[11px] text-slate-400 mt-1">
                Com <strong className="text-red-400">{settings.maxConsecutiveLosses ?? 2} stops seguidos</strong> você encerra
              </p>
            </div>

            <div
              className={`bg-slate-950/80 border rounded-xl p-3.5 transition-colors ${
                metrics.isDailyLossCountHit
                  ? 'border-red-500/80 bg-red-950/20'
                  : 'border-slate-800 focus-within:border-red-500/60'
              }`}
            >
              <label className="flex items-center justify-between text-xs font-medium text-slate-400 mb-1.5">
                <span>Qtd. Máx. de Stop no Dia</span>
                <ShieldAlert className="w-3.5 h-3.5 text-red-400" />
              </label>
              <div className="relative flex items-center">
                <input
                  type="number"
                  min="1"
                  max="50"
                  step="1"
                  value={settings.maxDailyLosses ?? 3}
                  onChange={(e) => handleSettingChange('maxDailyLosses', e.target.value)}
                  className="w-full bg-transparent text-lg font-bold text-red-400 focus:outline-none"
                />
                <span
                  className={`text-xs font-semibold whitespace-nowrap ${
                    metrics.isDailyLossCountHit ? 'text-red-400' : 'text-slate-400'
                  }`}
                >
                  hoje: {metrics.todayLossesCount}/{settings.maxDailyLosses ?? 3}
                </span>
              </div>
              <p className="text-[11px] text-slate-400 mt-1">
                Levou <strong className="text-red-400">{settings.maxDailyLosses ?? 3} loss no dia</strong>, encerra o pregão
              </p>
            </div>

            <div className="bg-slate-950/80 border border-slate-800 rounded-xl p-3.5">
              <label className="flex items-center justify-between text-xs font-medium text-slate-400 mb-1.5">
                <span>Taxa de Assertividade (Geral)</span>
                <Award className="w-3.5 h-3.5 text-emerald-400" />
              </label>
              <div className="text-lg font-bold text-emerald-400">
                {metrics.winRate}% <span className="text-xs font-normal text-slate-400">({metrics.winsCount}W / {metrics.lossesCount}L)</span>
              </div>
              <p className="text-[11px] text-slate-400 mt-1">Percentual de acerto histórico</p>
            </div>

            <div className="bg-slate-950/80 border border-slate-800 rounded-xl p-3.5">
              <label className="flex items-center justify-between text-xs font-medium text-slate-400 mb-1.5">
                <span>Total de Operações</span>
                <BarChart3 className="w-3.5 h-3.5 text-emerald-400" />
              </label>
              <div className="text-lg font-bold text-white">
                {metrics.totalTradesCount} <span className="text-xs font-normal text-slate-400">trades realizados</span>
              </div>
              <p className="text-[11px] text-slate-400 mt-1">Desde o início das anotações</p>
            </div>
          </div>
        </section>

        {/* --- SEÇÃO 2: GRÁFICO DE DESENVOLVIMENTO DA CONTA & RESULTADO MENSAL --- */}
        <section className="bg-slate-900/80 border border-slate-800 rounded-2xl p-5 shadow-xl space-y-6">
          <div className="flex flex-col md:flex-row md:items-center justify-between gap-4 pb-3 border-b border-slate-800">
            <div>
              <div className="flex items-center gap-2">
                <BarChart3 className="w-5 h-5 text-emerald-400" />
                <h2 className="text-base font-bold text-white">Desenvolvimento da Conta & Resultado</h2>
              </div>
              <p className="text-xs text-slate-400 mt-0.5">
                Acompanhe o crescimento do seu capital e o resultado consolidado do período
              </p>
            </div>

            <div className="flex flex-wrap items-center gap-3">
              <div className="flex items-center gap-2 bg-slate-950 border border-slate-800 rounded-xl px-3 py-1.5">
                <span className="text-xs text-slate-400">Resultado do Mês:</span>
                <span
                  className={`text-xs font-bold ${
                    currentMonthResult >= 0 ? 'text-emerald-400' : 'text-red-400'
                  }`}
                >
                  {currentMonthResult >= 0 ? '+' : ''}
                  {formatCurrency(currentMonthResult)}
                </span>
              </div>

              <div className="flex items-center gap-1 bg-slate-950 p-1 rounded-xl border border-slate-800">
                {[
                  { id: 'daily', label: 'Diário' },
                  { id: 'weekly', label: 'Semanal' },
                  { id: 'monthly', label: 'Mensal' }
                ].map((p) => (
                  <button
                    key={p.id}
                    onClick={() => setChartPeriod(p.id)}
                    className={`px-3 py-1 rounded-lg text-xs font-semibold transition-colors cursor-pointer ${
                      chartPeriod === p.id
                        ? 'bg-emerald-500 text-slate-950 shadow-md shadow-emerald-500/20'
                        : 'text-slate-400 hover:text-white'
                    }`}
                  >
                    {p.label}
                  </button>
                ))}
              </div>
            </div>
          </div>

          <div className="h-72 w-full">
            <ResponsiveContainer width="100%" height="100%">
              <ComposedChart data={chartData} margin={{ top: 10, right: 10, left: 10, bottom: 0 }}>
                <defs>
                  <linearGradient id="colorCapital" x1="0" y1="0" x2="0" y2="1">
                    <stop offset="5%" stopColor="#10b981" stopOpacity={0.35} />
                    <stop offset="95%" stopColor="#10b981" stopOpacity={0.0} />
                  </linearGradient>
                </defs>
                <CartesianGrid strokeDasharray="3 3" stroke="#1e293b" vertical={false} />
                <XAxis dataKey="label" stroke="#64748b" fontSize={11} tickLine={false} />
                <YAxis
                  stroke="#64748b"
                  fontSize={11}
                  tickLine={false}
                  axisLine={false}
                  tickFormatter={(val) => `R$ ${val >= 1000 ? (val / 1000).toFixed(0) + 'k' : val}`}
                  domain={['auto', 'auto']}
                />
                <Tooltip content={<CustomTooltip />} />
                <ReferenceLine
                  y={Number(settings.initialCapital) + Number(settings.monthlyTargetProfit)}
                  stroke="#10b981"
                  strokeDasharray="4 4"
                  label={{ value: 'Meta Mês', fill: '#10b981', fontSize: 11, position: 'top' }}
                />
                <Bar
                  dataKey="resultadoPeriodo"
                  name="Resultado Período"
                  radius={[4, 4, 0, 0]}
                  maxBarSize={36}
                >
                  {chartData.map((entry, index) => (
                    <Cell
                      key={`cell-${index}`}
                      fill={entry.resultadoPeriodo >= 0 ? '#10b981' : '#ef4444'}
                    />
                  ))}
                </Bar>
                <Area
                  type="monotone"
                  dataKey="capitalAcumulado"
                  name="Capital Acumulado"
                  stroke="#34d399"
                  strokeWidth={2.5}
                  fillOpacity={1}
                  fill="url(#colorCapital)"
                />
              </ComposedChart>
            </ResponsiveContainer>
          </div>
        </section>

        {/* --- SEÇÃO 3: CALENDÁRIO INTERATIVO DIA A DIA --- */}
        <section className="bg-slate-900/80 border border-slate-800 rounded-2xl p-5 shadow-xl space-y-4">
          <div className="flex flex-col sm:flex-row sm:items-center justify-between gap-3 pb-3 border-b border-slate-800">
            <div className="flex items-center gap-2">
              <Calendar className="w-5 h-5 text-emerald-400" />
              <h2 className="text-base font-bold text-white">Calendário de Resultados (Dia a Dia)</h2>
            </div>

            <div className="flex items-center gap-3">
              <span className="text-xs text-slate-400">Mês do Calendário:</span>
              <input
                type="month"
                value={calendarYearMonth}
                onChange={(e) => setCalendarYearMonth(e.target.value)}
                className="bg-slate-950 border border-slate-800 rounded-xl px-3 py-1.5 text-xs font-semibold text-white focus:outline-none focus:border-emerald-500"
              />
            </div>
          </div>

          <div className="grid grid-cols-7 gap-2 text-center">
            {['DOM', 'SEG', 'TER', 'QUA', 'QUI', 'SEX', 'SÁB'].map((d) => (
              <div key={d} className="text-[11px] font-bold text-slate-500 py-1">
                {d}
              </div>
            ))}

            {calendarDaysData.map((item, idx) => {
              if (item.day === null) {
                return <div key={`empty-${idx}`} className="h-20 bg-slate-950/30 rounded-xl border border-slate-900/50" />;
              }

              const hasTrades = item.data !== null && item.data.count > 0;
              const isPositive = hasTrades && item.data.pnl >= 0;
              const isNegative = hasTrades && item.data.pnl < 0;

              return (
                <div
                  key={`day-${item.day}`}
                  onClick={() => {
                    if (item.dateStr) setJournalDate(item.dateStr);
                  }}
                  className={`h-20 rounded-xl p-2 border flex flex-col justify-between transition-all cursor-pointer text-left ${
                    isPositive
                      ? 'bg-emerald-950/30 border-emerald-500/40 hover:border-emerald-500 shadow-sm shadow-emerald-500/10'
                      : isNegative
                      ? 'bg-red-950/30 border-red-500/40 hover:border-red-500 shadow-sm shadow-red-500/10'
                      : 'bg-slate-950/70 border-slate-800/80 hover:border-slate-700'
                  }`}
                >
                  <div className="flex items-center justify-between text-xs font-bold text-slate-400">
                    <span>{item.day}</span>
                    {hasTrades && (
                      <span className="text-[10px] text-slate-500 font-normal">
                        {item.data.count} op{item.data.count > 1 ? 's' : ''}
                      </span>
                    )}
                  </div>

                  {hasTrades ? (
                    <div>
                      <div
                        className={`text-xs sm:text-sm font-black ${
                          isPositive ? 'text-emerald-400' : 'text-red-400'
                        }`}
                      >
                        {isPositive ? '+' : ''}
                        {formatCurrency(item.data.pnl).replace('R$', '').trim()}
                      </div>
                      <div className="text-[10px] text-slate-400 flex items-center gap-1 mt-0.5">
                        <span className="text-emerald-400 font-semibold">{item.data.wins}W</span>
                        <span>•</span>
                        <span className="text-red-400 font-semibold">{item.data.losses}L</span>
                      </div>
                    </div>
                  ) : (
                    <span className="text-[10px] text-slate-600">—</span>
                  )}
                </div>
              );
            })}
          </div>
          <p className="text-[11px] text-slate-500 text-center pt-1">
            💡 Dica: Clique em qualquer dia do calendário para abrir instantaneamente o seu diário e ver as operações daquela data.
          </p>
        </section>

        {/* --- SEÇÃO 4: DIÁRIO DE TRADER DIÁRIO (PSICOLOGIA & ANOTAÇÕES DO DIA) --- */}
        <section className="bg-slate-900/80 border border-slate-800 rounded-2xl p-5 shadow-xl">
          <div className="flex flex-col md:flex-row md:items-center justify-between gap-4 mb-4 pb-3 border-b border-slate-800">
            <div className="flex items-center gap-2.5">
              <div className="w-8 h-8 rounded-lg bg-emerald-500/15 border border-emerald-500/30 flex items-center justify-center text-emerald-400">
                <BookOpen className="w-4 h-4" />
              </div>
              <div>
                <h2 className="text-base font-bold text-white flex items-center gap-2">
                  Diário de Trader — Anotações do Pregão
                  {journalSavedFeedback && (
                    <span className="text-[11px] font-medium text-emerald-400 bg-emerald-500/10 px-2 py-0.5 rounded-full">
                      Salvo automaticamente
                    </span>
                  )}
                </h2>
                <p className="text-xs text-slate-400">
                  Registre seu estado emocional, disciplina e observações gerais de cada dia operado
                </p>
              </div>
            </div>

            <div className="flex flex-wrap items-center gap-3">
              <div className="flex items-center gap-2 bg-slate-950 border border-slate-800 rounded-xl px-3 py-1.5">
                <Calendar className="w-3.5 h-3.5 text-emerald-400" />
                <input
                  type="date"
                  value={journalDate}
                  onChange={(e) => setJournalDate(e.target.value)}
                  className="bg-transparent text-xs font-semibold text-white focus:outline-none"
                />
              </div>

              <div className="flex items-center gap-2 text-xs bg-slate-950 border border-slate-800 rounded-xl px-3 py-1.5">
                <span className="text-slate-400">Dia:</span>
                <span
                  className={`font-bold ${
                    selectedDateStats.pnl > 0
                      ? 'text-emerald-400'
                      : selectedDateStats.pnl < 0
                      ? 'text-red-400'
                      : 'text-slate-300'
                  }`}
                >
                  {selectedDateStats.pnl > 0 ? '+' : ''}
                  {formatCurrency(selectedDateStats.pnl)}
                </span>
                <span className="text-slate-600">|</span>
                <span className="text-emerald-400 font-semibold">{selectedDateStats.wins}W</span>
                <span className="text-red-400 font-semibold">{selectedDateStats.losses}L</span>
              </div>
            </div>
          </div>

          <div className="grid grid-cols-1 lg:grid-cols-12 gap-4">
            <div className="lg:col-span-3 space-y-2">
              <label className="block text-xs font-medium text-slate-400">Estado Emocional / Humor</label>
              <div className="grid grid-cols-2 lg:grid-cols-1 gap-2">
                {[
                  { id: 'Focado', color: 'border-emerald-500/50 bg-emerald-500/15 text-emerald-300' },
                  { id: 'Confiante', color: 'border-emerald-500/50 bg-emerald-500/15 text-emerald-300' },
                  { id: 'Ansioso', color: 'border-amber-500/50 bg-amber-500/15 text-amber-300' },
                  { id: 'Dia de Fúria / Tilt', color: 'border-red-500/50 bg-red-500/15 text-red-300' }
                ].map((m) => (
                  <button
                    key={m.id}
                    type="button"
                    onClick={() => handleJournalUpdate('mood', m.id)}
                    className={`px-3 py-2 rounded-xl text-xs font-semibold text-left border transition-all cursor-pointer ${
                      currentJournal.mood === m.id
                        ? m.color
                        : 'bg-slate-950 border-slate-800 text-slate-400 hover:text-slate-200'
                    }`}
                  >
                    {m.id}
                  </button>
                ))}
              </div>
            </div>

            <div className="lg:col-span-9 flex flex-col">
              <label className="block text-xs font-medium text-slate-400 mb-2">
                Anotações do Dia ({formatDateBR(journalDate)})
              </label>
              <textarea
                rows={4}
                placeholder="Como foi o mercado hoje? Você seguiu seu gerenciamento de risco? Quais erros evitar amanhã?"
                value={currentJournal.content}
                onChange={(e) => handleJournalUpdate('content', e.target.value)}
                className="w-full flex-1 bg-slate-950 border border-slate-800 rounded-xl p-3.5 text-sm text-slate-200 placeholder:text-slate-600 focus:outline-none focus:border-emerald-500/60 resize-none leading-relaxed"
              />
            </div>
          </div>
        </section>

        {/* --- SEÇÃO 5: LANÇAMENTO DE OPERAÇÕES + HISTÓRICO EDITÁVEL --- */}
        <section className="grid grid-cols-1 lg:grid-cols-12 gap-6">
          <div
            id="trade-form-section"
            className={`lg:col-span-4 bg-slate-900/90 border rounded-2xl p-5 h-fit shadow-xl transition-all ${
              editingTradeId ? 'border-amber-500/60 ring-1 ring-amber-500/30' : 'border-slate-800'
            }`}
          >
            <div className="flex items-center justify-between mb-4 pb-3 border-b border-slate-800">
              <h2 className="text-sm font-bold uppercase tracking-wider text-white flex items-center gap-2">
                {editingTradeId ? (
                  <>
                    <Edit2 className="w-4 h-4 text-amber-400" />
                    Editar Operação
                  </>
                ) : (
                  <>
                    <Plus className="w-4 h-4 text-emerald-400" />
                    Registrar Operação
                  </>
                )}
              </h2>
              {editingTradeId && (
                <button
                  onClick={handleCancelEdit}
                  className="text-xs text-slate-400 hover:text-white flex items-center gap-1 cursor-pointer"
                >
                  <X className="w-3.5 h-3.5" /> Cancelar
                </button>
              )}
            </div>

            <form onSubmit={handleSaveTrade} className="space-y-4">
              <div>
                <label className="block text-xs font-medium text-slate-400 mb-1.5">Status da Operação (WIN / STOP)</label>
                <div className="grid grid-cols-2 gap-2.5">
                  <button
                    type="button"
                    onClick={() => setFormData((p) => ({ ...p, outcome: 'GAIN' }))}
                    className={`py-2.5 px-3 rounded-xl font-bold text-xs flex items-center justify-center gap-1.5 border transition-all cursor-pointer ${
                      formData.outcome === 'GAIN'
                        ? 'bg-emerald-500 text-slate-950 border-emerald-400 shadow-lg shadow-emerald-500/25'
                        : 'bg-slate-950 text-slate-400 border-slate-800 hover:border-emerald-500/40'
                    }`}
                  >
                    <TrendingUp className="w-4 h-4" />
                    WIN (Gain)
                  </button>

                  <button
                    type="button"
                    onClick={() => setFormData((p) => ({ ...p, outcome: 'LOSS' }))}
                    className={`py-2.5 px-3 rounded-xl font-bold text-xs flex items-center justify-center gap-1.5 border transition-all cursor-pointer ${
                      formData.outcome === 'LOSS'
                        ? 'bg-red-500 text-white border-red-400 shadow-lg shadow-red-500/25'
                        : 'bg-slate-950 text-slate-400 border-slate-800 hover:border-red-500/40'
                    }`}
                  >
                    <TrendingDown className="w-4 h-4" />
                    STOP (Loss)
                  </button>
                </div>
              </div>

              <div>
                <label className="block text-xs font-medium text-slate-400 mb-1.5">Resultado Financeiro (R$)</label>
                <div
                  className={`flex items-center bg-slate-950 border rounded-xl px-3.5 py-2.5 transition-colors ${
                    formData.outcome === 'GAIN'
                      ? 'border-slate-800 focus-within:border-emerald-500'
                      : 'border-slate-800 focus-within:border-red-500'
                  }`}
                >
                  <span
                    className={`font-bold text-sm mr-2 ${
                      formData.outcome === 'GAIN' ? 'text-emerald-400' : 'text-red-400'
                    }`}
                  >
                    {formData.outcome === 'GAIN' ? '+ R$' : '- R$'}
                  </span>
                  <input
                    type="number"
                    step="0.01"
                    min="0.01"
                    required
                    placeholder="0,00"
                    value={formData.amount}
                    onChange={(e) => setFormData((p) => ({ ...p, amount: e.target.value }))}
                    className="w-full bg-transparent text-base font-bold text-white focus:outline-none"
                  />
                </div>
              </div>

              <div className="grid grid-cols-2 gap-3">
                <div>
                  <label className="block text-xs font-medium text-slate-400 mb-1.5">Data</label>
                  <input
                    type="date"
                    required
                    value={formData.date}
                    onChange={(e) => setFormData((p) => ({ ...p, date: e.target.value }))}
                    className="w-full bg-slate-950 border border-slate-800 rounded-xl px-3 py-2 text-xs text-white focus:outline-none focus:border-emerald-500"
                  />
                </div>

                <div>
                  <label className="block text-xs font-medium text-slate-400 mb-1.5">Ativo</label>
                  <input
                    type="text"
                    required
                    placeholder="WIN, WDO, BTC..."
                    value={formData.asset}
                    onChange={(e) => setFormData((p) => ({ ...p, asset: e.target.value.toUpperCase() }))}
                    className="w-full bg-slate-950 border border-slate-800 rounded-xl px-3 py-2 text-xs font-bold text-white uppercase focus:outline-none focus:border-emerald-500"
                  />
                </div>
              </div>

              <div>
                <label className="flex items-center gap-1.5 text-xs font-medium text-slate-400 mb-1.5">
                  <Crosshair className="w-3.5 h-3.5 text-emerald-400" />
                  Valor da Entrada / Contratos
                </label>
                <input
                  type="text"
                  placeholder="Ex: 128.500 pts (5 cts) ou R$ 1.000"
                  value={formData.entryValue}
                  onChange={(e) => setFormData((p) => ({ ...p, entryValue: e.target.value }))}
                  className="w-full bg-slate-950 border border-slate-800 rounded-xl px-3.5 py-2 text-xs text-white focus:outline-none focus:border-emerald-500"
                />
              </div>

              <div>
                <label className="flex items-center gap-1.5 text-xs font-medium text-slate-400 mb-1.5">
                  <Layers className="w-3.5 h-3.5 text-emerald-400" />
                  Estratégia / Setup Utilizado
                </label>
                <input
                  type="text"
                  placeholder="Ex: Pullback VWAP, Rompimento Abertura..."
                  value={formData.strategy}
                  onChange={(e) => setFormData((p) => ({ ...p, strategy: e.target.value }))}
                  className="w-full bg-slate-950 border border-slate-800 rounded-xl px-3.5 py-2 text-xs text-white focus:outline-none focus:border-emerald-500"
                />
              </div>

              <div>
                <label className="block text-xs font-medium text-slate-400 mb-1.5">Direção</label>
                <div className="grid grid-cols-2 gap-2.5">
                  {['Compra', 'Venda'].map((dir) => (
                    <button
                      key={dir}
                      type="button"
                      onClick={() => setFormData((p) => ({ ...p, type: dir }))}
                      className={`py-2 rounded-xl text-xs font-semibold border transition-colors cursor-pointer ${
                        formData.type === dir
                          ? 'bg-slate-800 text-white border-slate-600'
                          : 'bg-slate-950 text-slate-500 border-slate-800 hover:text-slate-300'
                      }`}
                    >
                      {dir === 'Compra' ? 'Compra (Long)' : 'Venda (Short)'}
                    </button>
                  ))}
                </div>
              </div>

              <div>
                <label className="block text-xs font-medium text-slate-400 mb-1.5">Observações da Operação</label>
                <input
                  type="text"
                  placeholder="Detalhes da execução, gatilho, gerenciamento..."
                  value={formData.notes}
                  onChange={(e) => setFormData((p) => ({ ...p, notes: e.target.value }))}
                  className="w-full bg-slate-950 border border-slate-800 rounded-xl px-3.5 py-2 text-xs text-white focus:outline-none focus:border-emerald-500"
                />
              </div>

              <div>
                <label className="flex items-center gap-1.5 text-xs font-medium text-slate-400 mb-1.5">
                  <Camera className="w-3.5 h-3.5 text-emerald-400" />
                  Print da Operação (Gráfico)
                </label>

                {formData.screenshot ? (
                  <div className="relative rounded-xl overflow-hidden border border-slate-700 bg-slate-950 group">
                    <img
                      src={formData.screenshot}
                      alt="Print da operação"
                      className="w-full h-32 object-cover"
                    />
                    <div className="absolute inset-0 bg-slate-950/70 opacity-0 group-hover:opacity-100 transition-opacity flex items-center justify-center gap-2">
                      <button
                        type="button"
                        onClick={() => setFormData((p) => ({ ...p, screenshot: null }))}
                        className="px-3 py-1.5 rounded-lg bg-red-500 text-white text-xs font-semibold flex items-center gap-1 cursor-pointer"
                      >
                        <Trash2 className="w-3.5 h-3.5" /> Remover Print
                      </button>
                    </div>
                  </div>
                ) : (
                  <label className="flex flex-col items-center justify-center w-full h-24 border-2 border-dashed border-slate-800 hover:border-emerald-500/50 rounded-xl bg-slate-950/60 hover:bg-slate-950 transition-colors cursor-pointer">
                    <ImageIcon className="w-5 h-5 text-slate-500 mb-1" />
                    <span className="text-xs text-slate-400 font-medium">Clique para anexar print</span>
                    <span className="text-[10px] text-slate-600">PNG, JPG ou WEBP</span>
                    <input
                      type="file"
                      accept="image/*"
                      onChange={handleScreenshotUpload}
                      className="hidden"
                    />
                  </label>
                )}
              </div>

              <button
                type="submit"
                className={`w-full py-3 rounded-xl font-bold text-xs uppercase tracking-wider flex items-center justify-center gap-2 transition-all shadow-lg cursor-pointer ${
                  editingTradeId
                    ? 'bg-amber-500 hover:bg-amber-400 text-slate-950 shadow-amber-500/20'
                    : formData.outcome === 'GAIN'
                    ? 'bg-emerald-500 hover:bg-emerald-400 text-slate-950 shadow-emerald-500/25'
                    : 'bg-red-500 hover:bg-red-400 text-white shadow-red-500/25'
                }`}
              >
                {editingTradeId ? (
                  <>
                    <Check className="w-4 h-4" />
                    Salvar Alterações
                  </>
                ) : (
                  <>
                    <Plus className="w-4 h-4" />
                    Adicionar ao Histórico
                  </>
                )}
              </button>
            </form>
          </div>

          <div className="lg:col-span-8 bg-slate-900/80 border border-slate-800 rounded-2xl p-5 shadow-xl flex flex-col">
            <div className="flex flex-col sm:flex-row sm:items-center justify-between gap-3 mb-4 pb-3 border-b border-slate-800">
              <div>
                <h2 className="text-base font-bold text-white flex items-center gap-2">
                  <Calendar className="w-4 h-4 text-emerald-400" />
                  Histórico de Operações
                  <span className="text-xs font-normal text-slate-400">({filteredTrades.length})</span>
                </h2>
              </div>

              <div className="flex items-center gap-1.5 bg-slate-950 p-1 rounded-xl border border-slate-800 self-start sm:self-auto">
                {[
                  { id: 'ALL', label: 'Todos' },
                  { id: 'GAIN', label: 'WIN' },
                  { id: 'LOSS', label: 'STOP' }
                ].map((f) => (
                  <button
                    key={f.id}
                    onClick={() => setFilterType(f.id)}
                    className={`px-3 py-1 rounded-lg text-xs font-semibold transition-colors cursor-pointer ${
                      filterType === f.id
                        ? f.id === 'GAIN'
                          ? 'bg-emerald-500/20 text-emerald-400'
                          : f.id === 'LOSS'
                          ? 'bg-red-500/20 text-red-400'
                          : 'bg-slate-800 text-white'
                        : 'text-slate-400 hover:text-white'
                    }`}
                  >
                    {f.label}
                  </button>
                ))}
              </div>
            </div>

            {filteredTrades.length === 0 ? (
              <div className="flex-1 flex flex-col items-center justify-center py-12 text-center">
                <div className="w-12 h-12 rounded-full bg-slate-800/80 flex items-center justify-center mb-3 text-slate-500">
                  <BarChart3 className="w-6 h-6" />
                </div>
                <p className="text-sm font-medium text-slate-300">Nenhuma operação encontrada</p>
                <p className="text-xs text-slate-500 mt-1 max-w-xs">
                  Registre sua primeira operação no formulário ao lado para acompanhar seu desempenho.
                </p>
              </div>
            ) : (
              <div className="overflow-x-auto -mx-5 px-5">
                <table className="w-full text-left border-collapse">
                  <thead>
                    <tr className="border-b border-slate-800/80 text-[11px] font-semibold uppercase tracking-wider text-slate-400">
                      <th className="py-3 pr-3">Data</th>
                      <th className="py-3 px-3">Ativo / Entrada</th>
                      <th className="py-3 px-3 hidden md:table-cell">Estratégia & Obs.</th>
                      <th className="py-3 px-3 text-center">Print</th>
                      <th className="py-3 px-3 text-right">Status / Resultado</th>
                      <th className="py-3 pl-3 text-right">Ações</th>
                    </tr>
                  </thead>
                  <tbody className="divide-y divide-slate-800/60 text-xs">
                    {filteredTrades.map((trade) => {
                      const isGain = Number(trade.result) >= 0;
                      const isBeingEdited = editingTradeId === trade.id;
                      return (
                        <tr
                          key={trade.id}
                          className={`group transition-colors ${
                            isBeingEdited ? 'bg-amber-500/10' : 'hover:bg-slate-800/40'
                          }`}
                        >
                          <td className="py-3.5 pr-3 whitespace-nowrap text-slate-300 font-medium align-top">
                            <div>{formatDateBR(trade.date)}</div>
                            <button
                              type="button"
                              onClick={() => setJournalDate(trade.date)}
                              className="text-[10px] text-emerald-400/80 hover:underline mt-0.5 cursor-pointer"
                            >
                              Ver diário
                            </button>
                          </td>

                          <td className="py-3.5 px-3 whitespace-nowrap align-top">
                            <div className="flex items-center gap-2">
                              <span className="font-bold text-white bg-slate-800 px-2 py-0.5 rounded text-[11px]">
                                {trade.asset}
                              </span>
                              <span
                                className={`text-[11px] font-medium ${
                                  trade.type === 'Compra' ? 'text-emerald-400/90' : 'text-red-400/90'
                                }`}
                              >
                                {trade.type}
                              </span>
                            </div>
                            {trade.entryValue && (
                              <div className="text-[11px] text-slate-400 mt-1 font-mono">
                                Ent: {trade.entryValue}
                              </div>
                            )}
                          </td>

                          <td className="py-3.5 px-3 hidden md:table-cell align-top max-w-[220px]">
                            {trade.strategy && (
                              <div className="font-semibold text-slate-200 truncate">
                                {trade.strategy}
                              </div>
                            )}
                            <div className="text-slate-400 text-[11px] truncate mt-0.5">
                              {trade.notes || (!trade.strategy ? '—' : '')}
                            </div>
                          </td>

                          <td className="py-3.5 px-3 text-center align-top">
                            {trade.screenshot ? (
                              <button
                                type="button"
                                onClick={() => setPreviewTrade(trade)}
                                className="inline-flex items-center gap-1 px-2 py-1 rounded-lg bg-slate-800 hover:bg-emerald-500/20 text-emerald-400 border border-slate-700 hover:border-emerald-500/40 text-[11px] font-medium transition-colors cursor-pointer"
                                title="Ver print e detalhes"
                              >
                                <Eye className="w-3.5 h-3.5" />
                                <span className="hidden sm:inline">Print</span>
                              </button>
                            ) : (
                              <span className="text-slate-600 text-[11px]">—</span>
                            )}
                          </td>

                          <td className="py-3.5 px-3 whitespace-nowrap text-right align-top">
                            <div className="flex flex-col items-end gap-1">
                              <span
                                className={`px-2 py-0.5 rounded text-[10px] font-extrabold uppercase tracking-wider ${
                                  isGain
                                    ? 'bg-emerald-500 text-slate-950'
                                    : 'bg-red-500 text-white'
                                }`}
                              >
                                {isGain ? 'WIN' : 'STOP'}
                              </span>
                              <span
                                className={`font-bold text-xs ${
                                  isGain ? 'text-emerald-400' : 'text-red-400'
                                }`}
                              >
                                {isGain ? '+' : ''}
                                {formatCurrency(trade.result)}
                              </span>
                            </div>
                          </td>

                          <td className="py-3.5 pl-3 whitespace-nowrap text-right align-top">
                            <div className="inline-flex items-center gap-1">
                              <button
                                onClick={() => handleStartEdit(trade)}
                                className="p-2 rounded-lg text-slate-400 hover:text-amber-400 hover:bg-amber-500/10 transition-colors cursor-pointer"
                                title="Editar operação"
                              >
                                <Edit2 className="w-3.5 h-3.5" />
                              </button>
                              <button
                                onClick={() => handleDeleteTrade(trade.id)}
                                className="p-2 rounded-lg text-slate-400 hover:text-red-400 hover:bg-red-500/10 transition-colors cursor-pointer"
                                title="Excluir operação"
                              >
                                <Trash2 className="w-3.5 h-3.5" />
                              </button>
                            </div>
                          </td>
                        </tr>
                      );
                    })}
                  </tbody>
                </table>
              </div>
            )}
          </div>
        </section>
      </main>

      {/* --- MODAL DE VISUALIZAÇÃO DO PRINT E DETALHES DA OPERAÇÃO --- */}
      {previewTrade && (
        <div
          className="fixed inset-0 z-50 bg-slate-950/85 backdrop-blur-md flex items-center justify-center p-4"
          onClick={() => setPreviewTrade(null)}
        >
          <div
            className="bg-slate-900 border border-slate-700 rounded-2xl max-w-4xl w-full overflow-hidden shadow-2xl"
            onClick={(e) => e.stopPropagation()}
          >
            <div className="flex items-center justify-between px-5 py-3.5 border-b border-slate-800">
              <div className="flex items-center gap-2.5">
                <span
                  className={`px-2.5 py-0.5 rounded text-xs font-extrabold uppercase ${
                    Number(previewTrade.result) >= 0
                      ? 'bg-emerald-500 text-slate-950'
                      : 'bg-red-500 text-white'
                  }`}
                >
                  {Number(previewTrade.result) >= 0 ? 'WIN' : 'STOP'}
                </span>
                <span className="font-bold text-white text-sm">
                  {previewTrade.asset} • {previewTrade.type} ({formatDateBR(previewTrade.date)})
                </span>
                <span
                  className={`font-bold text-sm ${
                    Number(previewTrade.result) >= 0 ? 'text-emerald-400' : 'text-red-400'
                  }`}
                >
                  {Number(previewTrade.result) >= 0 ? '+' : ''}
                  {formatCurrency(previewTrade.result)}
                </span>
              </div>
              <button
                onClick={() => setPreviewTrade(null)}
                className="p-1.5 rounded-lg text-slate-400 hover:text-white hover:bg-slate-800 cursor-pointer"
              >
                <X className="w-5 h-5" />
              </button>
            </div>

            <div className="p-5 space-y-4 max-h-[80vh] overflow-y-auto">
              {previewTrade.screenshot && (
                <div className="rounded-xl overflow-hidden border border-slate-800 bg-slate-950 flex justify-center">
                  <img
                    src={previewTrade.screenshot}
                    alt="Print da operação"
                    className="max-h-[55vh] w-auto object-contain"
                  />
                </div>
              )}

              <div className="grid grid-cols-1 sm:grid-cols-2 gap-3 text-xs">
                <div className="bg-slate-950/80 border border-slate-800 rounded-xl p-3">
                  <span className="text-slate-400 block mb-1">Estratégia / Setup:</span>
                  <span className="font-semibold text-white">
                    {previewTrade.strategy || 'Não informada'}
                  </span>
                </div>
                <div className="bg-slate-950/80 border border-slate-800 rounded-xl p-3">
                  <span className="text-slate-400 block mb-1">Valor da Entrada / Lote:</span>
                  <span className="font-semibold text-white font-mono">
                    {previewTrade.entryValue || 'Não informado'}
                  </span>
                </div>
              </div>

              {previewTrade.notes && (
                <div className="bg-slate-950/80 border border-slate-800 rounded-xl p-3 text-xs">
                  <span className="text-slate-400 block mb-1">Observações:</span>
                  <p className="text-slate-200">{previewTrade.notes}</p>
                </div>
              )}
            </div>
          </div>
        </div>
      )}
    </div>
  );
}