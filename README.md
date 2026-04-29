import React, { useState, useEffect } from 'react';
import { motion, AnimatePresence } from 'motion/react';
import { 
  Plus, 
  Search, 
  Calendar, 
  User, 
  Layout, 
  CheckCircle2, 
  Edit2, 
  Trash2,
  Filter,
  PieChart as PieChartIcon,
  BarChart as BarChartIcon,
  X
} from 'lucide-react';
import moment from 'moment';
import { 
  BarChart, 
  Bar, 
  XAxis, 
  YAxis, 
  CartesianGrid, 
  Tooltip, 
  ResponsiveContainer,
  PieChart,
  Pie,
  Cell,
  Legend
} from 'recharts';

const AREAS = [
  'EHS', 
  'SODEXO', 
  'INHAUS', 
  'OBRA CIVIL', 
  'SEGURANÇA', 
  'PROJETO', 
  'AVARIA SODEXO', 
  'AVARIA IN HAUS', 
  'AVARIA TRANSPORTADORA', 
  'REFORMA', 
  'PINTURA',
  'OUTROS'
];
const REQ_STATUS = ['REQUISIÇÃO', 'PEDIDO DE COMPRA'];

type TaskStatus = 'Não iniciado' | 'Em andamento' | 'Concluído';

interface Task {
  id: string;
  titulo: string;
  responsavel: string;
  area: string;
  data_inicio: string;
  data_conclusao: string;
  status: TaskStatus;
  status_requisicao: string;
  observacao: string;
}

export default function App() {
  const [planos, setPlanos] = useState<Task[]>(() => {
    const saved = localStorage.getItem('planos_v24');
    return saved ? JSON.parse(saved) : [];
  });

  const [activeTab, setActiveTab] = useState<'lista' | 'concluidos' | 'dashboard'>('lista');
  const [searchTerm, setSearchTerm] = useState('');
  const [filtroArea, setFiltroArea] = useState('TODOS');
  const [filtroReqStatus, setFiltroReqStatus] = useState('TODOS');
  const [filtroResponsavel, setFiltroResponsavel] = useState('TODOS');
  
  const [isDrawerOpen, setIsDrawerOpen] = useState(false);
  const [gestorAutenticado, setGestorAutenticado] = useState(false);
  const [editingTask, setEditingTask] = useState<Task | null>(null);

  const [newAction, setNewAction] = useState({
    titulo: '',
    responsavel: '',
    area: 'OUTROS',
    data_inicio: moment().format('YYYY-MM-DD'),
    data_conclusao: moment().add(7, 'days').format('YYYY-MM-DD'),
    status_requisicao: '',
    observacao: ''
  });

  useEffect(() => {
    localStorage.setItem('planos_v24', JSON.stringify(planos));
  }, [planos]);

  const handleCreate = (e: React.FormEvent) => {
    e.preventDefault();
    const task: Task = {
      ...newAction,
      id: Date.now().toString(),
      status: 'Não iniciado'
    };
    setPlanos([task, ...planos]);
    setNewAction({
      titulo: '',
      responsavel: '',
      area: 'OUTROS',
      data_inicio: moment().format('YYYY-MM-DD'),
      data_conclusao: moment().add(7, 'days').format('YYYY-MM-DD'),
      status_requisicao: '',
      observacao: ''
    });
    setIsDrawerOpen(false);
  };

  const updateStatus = (id: string, status: TaskStatus) => {
    setPlanos(planos.map(p => p.id === id ? { ...p, status } : p));
  };

  const updateObservacao = (id: string, observacao: string) => {
    setPlanos(planos.map(p => p.id === id ? { ...p, observacao } : p));
  };

  const handleDelete = (id: string) => {
    if (window.confirm('Excluir esta ação permanentemente?')) {
      setPlanos(planos.filter(p => p.id !== id));
      setEditingTask(null);
    }
  };

  const handleUpdate = (e: React.FormEvent) => {
    e.preventDefault();
    if (!editingTask) return;
    setPlanos(planos.map(p => p.id === editingTask.id ? editingTask : p));
    setEditingTask(null);
  };

  const getStats = () => {
    const total = planos.length;
    const concluido = planos.filter(p => p.status === 'Concluído').length;
    const andamento = planos.filter(p => p.status === 'Em andamento').length;
    const naoIniciado = planos.filter(p => p.status === 'Não iniciado').length;
    return { total, concluido, andamento, naoIniciado };
  };

  const filteredPlanos = planos.filter(p => {
    const matchesSearch = p.titulo.toLowerCase().includes(searchTerm.toLowerCase()) || 
                         p.responsavel.toLowerCase().includes(searchTerm.toLowerCase());
    const matchesArea = filtroArea === 'TODOS' || p.area === filtroArea;
    const matchesReq = filtroReqStatus === 'TODOS' || p.status_requisicao === filtroReqStatus;
    const matchesResp = filtroResponsavel === 'TODOS' || p.responsavel === filtroResponsavel;
    const matchesTab = activeTab === 'concluidos' ? p.status === 'Concluído' : p.status !== 'Concluído';
    
    return matchesSearch && matchesArea && matchesReq && matchesResp && matchesTab;
  });

  return (
    <div className="min-h-screen bg-[#F8FAFC] text-slate-900 font-sans selection:bg-blue-100 italic-none">
      {/* HEADER TOP */}
      <header className="bg-white border-b border-slate-200 sticky top-0 z-40">
        <div className="max-w-7xl mx-auto px-4 md:px-8">
          <div className="h-20 flex items-center justify-between">
            <div className="flex items-center gap-3">
              <div className="w-10 h-10 bg-blue-600 rounded-xl flex items-center justify-center shadow-lg shadow-blue-200">
                <Layout className="text-white" size={20} />
              </div>
              <span className="font-black text-xl tracking-tighter uppercase">
                EXECUTE PRO <span className="text-blue-600">v24</span>
              </span>
            </div>
            <div className="flex gap-4 md:gap-6 text-[12px] md:text-sm items-center">
              <button 
                onClick={() => setActiveTab('lista')} 
                className={`pb-1 font-black uppercase transition-all ${activeTab === 'lista' ? 'text-blue-600 border-b-2 border-blue-600' : 'text-slate-400 hover:text-slate-600'}`}
              >
                PROJETOS
              </button>
              <button 
                onClick={() => setActiveTab('concluidos')} 
                className={`pb-1 font-black uppercase transition-all ${activeTab === 'concluidos' ? 'text-blue-600 border-b-2 border-blue-600' : 'text-slate-400 hover:text-slate-600'}`}
              >
                CONCLUÍDOS
              </button>
              <button 
                onClick={() => setActiveTab('dashboard')} 
                className={`pb-1 font-black uppercase transition-all ${activeTab === 'dashboard' ? 'text-blue-600 border-b-2 border-blue-600' : 'text-slate-400 hover:text-slate-600'}`}
              >
                BI / DASHBOARD
              </button>
              <div className="h-8 w-[1px] bg-slate-200 mx-2 hidden md:block"></div>
              <button 
                onClick={() => setGestorAutenticado(!gestorAutenticado)}
                className={`px-4 py-2 rounded-full font-black text-[10px] uppercase transition-all ${gestorAutenticado ? 'bg-blue-600 text-white shadow-md' : 'bg-slate-100 text-slate-500 hover:bg-slate-200'}`}
              >
                {gestorAutenticado ? 'MODO GESTOR ON' : 'ACESSO GESTOR'}
              </button>
            </div>
          </div>
        </div>
      </header>

      <main className="max-w-7xl mx-auto px-4 md:px-8 py-8 md:py-12">
        
        {/* FILTERS SECTION */}
        <section className="bg-white p-6 rounded-2xl border border-slate-200 shadow-sm mb-8">
          <div className="grid grid-cols-1 md:grid-cols-4 gap-6 items-end">
            <div className="flex flex-col gap-2">
              <label className="text-[11px] font-black text-slate-500 uppercase">ÁREA / SETOR</label>
              <select 
                value={filtroArea}
                onChange={(e) => setFiltroArea(e.target.value)}
                className="bg-slate-50 border-2 border-slate-100 p-3 rounded-xl text-sm font-black outline-none w-full uppercase cursor-pointer"
              >
                <option value="TODOS">MOSTRAR TODAS</option>
                {AREAS.map(a => <option key={a} value={a}>{a}</option>)}
              </select>
            </div>

            <div className="flex flex-col gap-2">
              <label className="text-[11px] font-black text-slate-500 uppercase">STATUS</label>
              <select 
                value={filtroReqStatus}
                onChange={(e) => setFiltroReqStatus(e.target.value)}
                className="bg-slate-50 border-2 border-slate-100 p-3 rounded-xl text-sm font-black outline-none w-full uppercase cursor-pointer"
              >
                <option value="TODOS">TODOS OS TIPOS</option>
                {REQ_STATUS.map(rs => <option key={rs} value={rs}>{rs}</option>)}
              </select>
            </div>

            <div className="flex flex-col gap-2">
              <label className="text-[11px] font-black text-slate-500 uppercase">RESPONSÁVEL</label>
              <select 
                value={filtroResponsavel}
                onChange={(e) => setFiltroResponsavel(e.target.value)}
                className="bg-slate-50 border-2 border-slate-100 p-3 rounded-xl text-sm font-black outline-none w-full uppercase cursor-pointer"
              >
                <option value="TODOS">TODOS OS NOMES</option>
                {[...new Set(planos.map(p => p.responsavel))].sort().map(n => (
                  <option key={n} value={n}>{n}</option>
                ))}
              </select>
            </div>

            <div>
              <button 
                onClick={() => setIsDrawerOpen(true)}
                className="w-full bg-blue-600 text-white h-[46px] rounded-xl font-black shadow-lg hover:bg-blue-700 transition uppercase text-sm flex items-center justify-center gap-2"
              >
                <Plus size={16} />
                NOVA AÇÃO PROJETO
              </button>
            </div>
          </div>
        </section>

        <div className="grid grid-cols-12 gap-8">
          {/* KPI SIDEBAR */}
          <div className="col-span-12 lg:col-span-3 flex flex-col gap-4">
            <div className="bg-white p-6 rounded-2xl border border-slate-200 shadow-sm">
              <p className="text-xs font-black text-slate-400 uppercase mb-1">TOTAL PENDENTE</p>
              <h4 className="text-4xl font-black text-slate-900 uppercase tracking-tighter">
                {planos.filter(p => p.status !== 'Concluído').length}
              </h4>
            </div>
            <div className="bg-red-50 p-6 rounded-2xl border border-red-100 shadow-sm">
              <p className="text-xs font-black text-red-400 uppercase mb-1">EM ATRASO</p>
              <h4 className="text-4xl font-black text-red-600 uppercase tracking-tighter">
                {planos.filter(p => {
                  const prazo = moment(p.data_conclusao).startOf('day');
                  return p.status !== 'Concluído' && moment().isAfter(prazo);
                }).length}
              </h4>
            </div>
            <div className="bg-green-50 p-6 rounded-2xl border border-green-100 shadow-sm">
              <p className="text-xs font-black text-green-400 uppercase mb-1">CONCLUÍDO (GERAL)</p>
              <h4 className="text-4xl font-black text-green-600 uppercase tracking-tighter">
                {planos.filter(p => p.status === 'Concluído').length}
              </h4>
            </div>
          </div>

          {/* MAIN CONTENT */}
          <div className="col-span-12 lg:col-span-9 space-y-6">
            
            {/* KPI POR ÁREA NOS CONCLUÍDOS */}
            {activeTab === 'concluidos' && (
              <motion.div 
                initial={{ opacity: 0, y: -10 }}
                animate={{ opacity: 1, y: 0 }}
                className="grid grid-cols-2 md:grid-cols-4 gap-3 mb-6"
              >
                {AREAS.map(area => {
                  const count = planos.filter(p => p.status === 'Concluído' && p.area === area).length;
                  if (count === 0) return null;
                  return (
                    <div key={area} className="bg-white p-3 rounded-xl border border-slate-200 shadow-sm flex flex-col items-center justify-center text-center">
                      <span className="text-xs font-black text-slate-400 uppercase mb-1">{area}</span>
                      <span className="text-xl font-black text-green-600">{count}</span>
                    </div>
                  );
                })}
              </motion.div>
            )}

            {activeTab === 'dashboard' ? (
              <div className="space-y-8">
                {/* CHARTS GRID */}
                <div className="grid grid-cols-1 lg:grid-cols-2 gap-8">
                  <div className="bg-white p-6 rounded-2xl border border-slate-200 shadow-sm flex flex-col h-[400px]">
                    <div className="flex items-center gap-2 mb-6">
                      <BarChartIcon className="text-blue-600" size={20} />
                      <h3 className="text-sm font-black text-slate-800 uppercase tracking-tighter">DISTRIBUIÇÃO POR STATUS</h3>
                    </div>
                    <div className="flex-1 w-full">
                      <ResponsiveContainer width="100%" height="100%">
                        <BarChart data={[
                          { name: 'PENDENTE', value: getStats().naoIniciado },
                          { name: 'CURSO', value: getStats().andamento },
                          { name: 'PRONTO', value: getStats().concluido }
                        ]}>
                          <CartesianGrid strokeDasharray="3 3" vertical={false} stroke="#f1f5f9" />
                          <XAxis dataKey="name" axisLine={false} tickLine={false} tick={{ fontSize: 10, fontWeight: 900, fill: '#64748b' }} />
                          <YAxis axisLine={false} tickLine={false} tick={{ fontSize: 10, fontWeight: 900, fill: '#64748b' }} />
                          <Tooltip contentStyle={{ borderRadius: '12px', border: 'none', boxShadow: '0 10px 15px -3px rgba(0, 0, 0, 0.1)', fontSize: '12px', fontWeight: 900 }} />
                          <Bar dataKey="value" radius={[8, 8, 0, 0]}>
                            <Cell fill="#2563eb" /><Cell fill="#facc15" /><Cell fill="#16a34a" />
                          </Bar>
                        </BarChart>
                      </ResponsiveContainer>
                    </div>
                  </div>

                  <div className="bg-white p-6 rounded-2xl border border-slate-200 shadow-sm flex flex-col h-[400px]">
                    <div className="flex items-center gap-2 mb-6">
                      <PieChartIcon className="text-blue-600" size={20} />
                      <h3 className="text-sm font-black text-slate-800 uppercase tracking-tighter">BREAKDOWN POR ÁREA</h3>
                    </div>
                    <div className="flex-1 w-full">
                      <ResponsiveContainer width="100%" height="100%">
                        <PieChart>
                          <Pie
                            data={AREAS.map((area, idx) => ({
                              name: area,
                              value: planos.filter(p => p.area === area).length
                            })).filter(d => d.value > 0)}
                            cx="50%" cy="50%" innerRadius={60} outerRadius={80} paddingAngle={5} dataKey="value"
                          >
                            {AREAS.map((_, index) => (
                              <Cell key={`cell-${index}`} fill={['#2563eb', '#16a34a', '#facc15', '#dc2626', '#8b5cf6', '#ec4899', '#f97316', '#06b6d4'][index % 8]} />
                            ))}
                          </Pie>
                          <Tooltip contentStyle={{ borderRadius: '12px', border: 'none', boxShadow: '0 10px 15px -3px rgba(0, 0, 0, 0.1)', fontSize: '11px', fontWeight: 900 }} />
                          <Legend layout="vertical" align="right" verticalAlign="middle" wrapperStyle={{ fontSize: '10px', fontWeight: 900 }} />
                        </PieChart>
                      </ResponsiveContainer>
                    </div>
                  </div>
                </div>

                {/* BENTO GRID SUMMARY */}
                <div className="grid grid-cols-1 md:grid-cols-3 gap-6">
                  <div className="bg-white p-6 rounded-2xl border-l-8 border-blue-600 shadow-sm">
                    <h4 className="text-[10px] font-black text-slate-400 uppercase mb-4">TAXA DE SUCESSO</h4>
                    <div className="text-3xl font-black text-slate-900 tracking-tighter">
                      {Math.round((getStats().concluido / (getStats().total || 1)) * 100)}%
                    </div>
                  </div>
                </div>
              </div>
            ) : (
              <div className="space-y-4">
                <AnimatePresence>
                  {filteredPlanos.map(p => {
                    const prazo = moment(p.data_conclusao);
                    const diff = prazo.diff(moment(), 'days');
                    const statusLabel = p.status === 'Concluído' ? 'CONCLUÍDO' : diff < 0 ? 'ATRASADO' : diff === 0 ? 'FINALIZA HOJE' : `FALTAM ${diff} DIAS`;

                    return (
                      <motion.div 
                        key={p.id}
                        layout
                        initial={{ opacity: 0, x: -20 }}
                        animate={{ opacity: 1, x: 0 }}
                        exit={{ opacity: 0, x: 20 }}
                        className="bg-white hover:bg-slate-50 transition p-6 rounded-2xl border border-slate-200 shadow-sm flex flex-col md:flex-row gap-6 items-start md:items-center relative"
                      >
                        <div className="flex-grow w-full">
                          <div className="flex justify-between items-start mb-2">
                            <span className={`${diff < 0 && p.status !== 'Concluído' ? 'bg-red-100 text-red-700 italic' : p.status === 'Concluído' ? 'bg-green-100 text-green-700' : 'bg-blue-100 text-blue-700'} text-[10px] px-2 py-0.5 rounded font-black uppercase`}>
                              {statusLabel}
                            </span>
                            <span className="text-slate-400 text-[11px] font-black uppercase">PRAZO: {moment(p.data_conclusao).format('DD/MM/YYYY')}</span>
                          </div>
                          <h3 className="text-base font-black text-slate-800 uppercase mb-3">{p.titulo}</h3>
                          <div className="flex items-center gap-2 mb-3">
                            <div className="h-2 w-2 rounded-full bg-slate-300"></div>
                            <span className="text-[11px] font-black text-slate-500 uppercase">{p.responsavel}</span>
                          </div>
                          <textarea 
                            defaultValue={p.observacao}
                            onBlur={(e) => updateObservacao(p.id, e.target.value)}
                            className="w-full p-3 bg-slate-50 border border-slate-100 rounded-lg text-xs font-black text-slate-600 outline-none focus:bg-white transition" 
                            placeholder="NOTAS E PENDÊNCIAS..."
                          />
                        </div>
                        <div className="md:w-52 w-full flex flex-col gap-3">
                          <select 
                            value={p.status}
                            onChange={(e) => updateStatus(p.id, e.target.value as TaskStatus)}
                            className="text-xs font-black p-3 border border-slate-200 rounded-lg uppercase bg-white cursor-pointer outline-none focus:border-blue-600"
                          >
                            <option value="Não iniciado">Não iniciado</option>
                            <option value="Em andamento">Em andamento</option>
                            <option value="Concluído">Concluído</option>
                          </select>
                          <div className="flex justify-between items-center gap-4">
                            <span className="text-[10px] font-black text-slate-400 bg-slate-50 px-2 py-1 rounded">
                              {p.status_requisicao ? p.status_requisicao : 'S/ DOCUMENTO'}
                            </span>
                            {gestorAutenticado && (
                              <button 
                                onClick={() => setEditingTask(p)}
                                className="p-2 text-slate-400 hover:text-blue-600 hover:bg-blue-50 rounded-lg transition"
                                title="Editar Projeto"
                              >
                                <Edit2 size={16} />
                              </button>
                            )}
                          </div>
                        </div>
                      </motion.div>
                    );
                  })}
                </AnimatePresence>
                {filteredPlanos.length === 0 && (
                  <div className="py-20 text-center text-slate-400 font-black uppercase text-xs">
                    Nenhuma ação encontrada para este filtro.
                  </div>
                )}
              </div>
            )}
          </div>
        </div>
      </main>

      {/* DRAWER NOVA TAREFA */}
      <AnimatePresence>
        {isDrawerOpen && (
          <div className="fixed inset-0 z-50 flex justify-end">
            <motion.div 
              initial={{ opacity: 0 }} animate={{ opacity: 1 }} exit={{ opacity: 0 }}
              onClick={() => setIsDrawerOpen(false)}
              className="absolute inset-0 bg-slate-900/40 backdrop-blur-sm"
            />
            <motion.div 
              initial={{ x: '100%' }} animate={{ x: 0 }} exit={{ x: '100%' }}
              className="relative w-full max-w-md bg-white h-full shadow-2xl p-8 overflow-y-auto"
            >
              <div className="flex justify-between items-center mb-8">
                <h2 className="text-xl font-black tracking-tighter uppercase">NOVA AÇÃO PROJETO</h2>
                <button onClick={() => setIsDrawerOpen(false)} className="p-2 hover:bg-slate-100 rounded-full">
                  <X size={20} />
                </button>
              </div>
              <form onSubmit={handleCreate} className="space-y-6">
                <div>
                  <label className="text-xs text-slate-400 mb-1 block font-black uppercase">TÍTULO DA TAREFA</label>
                  <input type="text" required value={newAction.titulo} onChange={(e) => setNewAction({...newAction, titulo: e.target.value})} className="w-full p-4 border border-slate-200 rounded-xl text-sm font-black bg-slate-50 uppercase" />
                </div>
                <div>
                  <label className="text-xs text-slate-400 mb-1 block font-black uppercase">RESPONSÁVEL</label>
                  <input type="text" required value={newAction.responsavel} onChange={(e) => setNewAction({...newAction, responsavel: e.target.value})} className="w-full p-4 border border-slate-200 rounded-xl text-sm font-black bg-slate-50 uppercase" />
                </div>
                <div className="grid grid-cols-2 gap-4">
                  <div>
                    <label className="text-xs text-slate-400 mb-1 block font-black uppercase">ÁREA</label>
                    <select required value={newAction.area} onChange={(e) => setNewAction({...newAction, area: e.target.value})} className="w-full p-4 border border-slate-200 rounded-xl text-sm font-black bg-slate-50">
                      {AREAS.map(a => <option key={a} value={a}>{a}</option>)}
                    </select>
                  </div>
                  <div>
                    <label className="text-xs text-slate-400 mb-1 block font-black uppercase">STATUS REQ.</label>
                    <select value={newAction.status_requisicao} onChange={(e) => setNewAction({...newAction, status_requisicao: e.target.value})} className="w-full p-4 border border-slate-200 rounded-xl text-sm font-black bg-slate-50">
                      <option value="">NÃO INFORMADO</option>
                      {REQ_STATUS.map(rs => <option key={rs} value={rs}>{rs}</option>)}
                    </select>
                  </div>
                </div>
                <div className="grid grid-cols-2 gap-4">
                  <div>
                    <label className="text-xs text-slate-400 mb-1 block font-black uppercase">INÍCIO</label>
                    <input type="date" required value={newAction.data_inicio} onChange={(e) => setNewAction({...newAction, data_inicio: e.target.value})} className="w-full p-4 border border-slate-200 rounded-xl text-sm font-black bg-slate-50" />
                  </div>
                  <div>
                    <label className="text-xs text-slate-400 mb-1 block font-black uppercase">CONCLUSÃO</label>
                    <input type="date" required value={newAction.data_conclusao} onChange={(e) => setNewAction({...newAction, data_conclusao: e.target.value})} className="w-full p-4 border border-slate-200 rounded-xl text-sm font-black bg-slate-50" />
                  </div>
                </div>
                <button type="submit" className="w-full bg-blue-600 text-white p-5 rounded-xl font-black uppercase shadow-lg shadow-blue-100 hover:bg-blue-700 transition tracking-widest mt-6">CRIAR PLANO DE AÇÃO</button>
              </form>
            </motion.div>
          </div>
        )}
      </AnimatePresence>

      {/* MODAL EDIÇÃO */}
      {editingTask && (
        <div className="fixed inset-0 z-50 flex items-center justify-center p-4">
          <div className="absolute inset-0 bg-slate-900/60 backdrop-blur-sm" onClick={() => setEditingTask(null)}></div>
          <motion.div initial={{ scale: 0.95, opacity: 0 }} animate={{ scale: 1, opacity: 1 }} className="relative bg-white rounded-3xl w-full max-w-lg p-8 shadow-2xl overflow-hidden">
            <h3 className="text-xl font-black mb-6 uppercase tracking-tighter">ALTERAR PROJETO</h3>
            <form onSubmit={handleUpdate} className="space-y-4">
              <div>
                <label className="text-xs text-slate-400 mb-1 block font-black uppercase">TÍTULO</label>
                <input type="text" value={editingTask.titulo} onChange={(e) => setEditingTask({...editingTask, titulo: e.target.value})} className="w-full p-4 bg-slate-50 border border-slate-100 rounded-xl text-sm font-black uppercase" />
              </div>
              <div className="grid grid-cols-2 gap-4">
                <div>
                  <label className="text-xs text-slate-400 mb-1 block font-black uppercase">RESPONSÁVEL</label>
                  <input type="text" value={editingTask.responsavel} onChange={(e) => setEditingTask({...editingTask, responsavel: e.target.value})} className="w-full p-4 bg-slate-50 border border-slate-100 rounded-xl text-sm font-black uppercase" />
                </div>
                <div>
                  <label className="text-xs text-slate-400 mb-1 block font-black uppercase">ÁREA</label>
                  <select value={editingTask.area} onChange={(e) => setEditingTask({...editingTask, area: e.target.value})} className="w-full p-4 bg-slate-50 border border-slate-100 rounded-xl text-sm font-black">
                    {AREAS.map(a => <option key={a} value={a}>{a}</option>)}
                  </select>
                </div>
              </div>
              <div className="grid grid-cols-2 gap-4">
                <div>
                  <label className="text-xs text-slate-400 mb-1 block font-black uppercase">INÍCIO</label>
                  <input type="date" value={editingTask.data_inicio} onChange={(e) => setEditingTask({...editingTask, data_inicio: e.target.value})} className="w-full p-4 bg-slate-50 border border-slate-100 rounded-xl text-sm font-black" />
                </div>
                <div>
                  <label className="text-xs text-slate-400 mb-1 block font-black uppercase">FIM</label>
                  <input type="date" value={editingTask.data_conclusao} onChange={(e) => setEditingTask({...editingTask, data_conclusao: e.target.value})} className="w-full p-4 bg-slate-50 border border-slate-100 rounded-xl text-sm font-black" />
                </div>
              </div>
              <div className="flex gap-4 mt-8">
                <button type="submit" className="flex-1 bg-blue-600 text-white py-4 rounded-xl font-black uppercase shadow-lg shadow-blue-100">SALVAR TUDO</button>
                <button type="button" onClick={() => setEditingTask(null)} className="px-6 py-4 bg-slate-100 text-slate-500 rounded-xl font-black uppercase">VOLTAR</button>
              </div>
              <button type="button" onClick={() => handleDelete(editingTask.id)} className="w-full mt-4 bg-red-50 text-red-600 py-3 rounded-xl font-black text-xs border border-red-100 hover:bg-red-100 transition uppercase tracking-widest">EXCLUIR DEFINITIVAMENTE</button>
            </form>
          </motion.div>
        </div>
      )}
    </div>
  );
}
