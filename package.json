import React, { useEffect, useMemo, useState } from "react";
import { supabase } from "./lib/supabase";

const LOGO_URL = "/Logo_dubworks.png";
const USERS_STORAGE_KEY = "dubworks_users_v1";

type Cargo = "diretoria" | "lider" | "editor";

type Usuario = {
  nome: string;
  login: string;
  senha: string;
  cargo: Cargo;
  vinculo: string;
};

type ElencoItem = {
  id: string;
  personagem: string;
  dublador: string;
  funcao: string;
};

type Projeto = {
  ID: string;
  Projeto: string;
  Tipo: string;
  Genero: string;
  Prioridade: string;
  Dupla: string;
  Lider: string;
  Telefone_Lider: string;
  Editor: string;
  Telefone_Editor: string;
  Status: string;
  Data_Inicio: string;
  Video_Editor_Link: string;
  Registro_Semanal: string;
  Observacoes: string;
  Elenco: ElencoItem[];
};

const usuariosPadrao: Usuario[] = [
  {
    nome: "Lorrane",
    login: "diretoria",
    senha: "1234",
    cargo: "diretoria",
    vinculo: "todos",
  },
];

const projetoVazio: Projeto = {
  ID: "",
  Projeto: "",
  Tipo: "Projeto",
  Genero: "",
  Prioridade: "P0",
  Dupla: "",
  Lider: "",
  Telefone_Lider: "",
  Editor: "",
  Telefone_Editor: "",
  Status: "Planejamento",
  Data_Inicio: "",
  Video_Editor_Link: "",
  Registro_Semanal: "",
  Observacoes: "",
  Elenco: [],
};

const elencoVazio: ElencoItem = {
  id: "",
  personagem: "",
  dublador: "",
  funcao: "",
};

const estilos = {
  azul: "#175cd3",
  azulEscuro: "#0d47a1",
  azulClaro: "#eef5ff",
  borda: "#d7e4f7",
  texto: "#183153",
  textoSuave: "#667085",
  fundo: "#f4f8fd",
  branco: "#ffffff",
};

function normalizar(valor?: string) {
  return (valor || "").trim().toLowerCase();
}

function carregarUsuariosLocal(): Usuario[] {
  const raw = localStorage.getItem(USERS_STORAGE_KEY);
  if (!raw) return [];
  try {
    return JSON.parse(raw);
  } catch {
    return [];
  }
}

function salvarUsuariosLocal(lista: Usuario[]) {
  localStorage.setItem(USERS_STORAGE_KEY, JSON.stringify(lista));
}

function podeVerProjeto(usuario: Usuario | null, projeto: Projeto) {
  if (!usuario) return false;
  if (usuario.cargo === "diretoria") return true;
  if (usuario.cargo === "lider") {
    return normalizar(projeto.Lider) === normalizar(usuario.vinculo);
  }
  if (usuario.cargo === "editor") {
    return normalizar(projeto.Editor) === normalizar(usuario.vinculo);
  }
  return false;
}

function podeEditarProjeto(usuario: Usuario | null, projeto: Projeto | null) {
  if (!usuario || !projeto) return false;
  if (usuario.cargo === "diretoria") return true;
  if (usuario.cargo === "lider") {
    return normalizar(projeto.Lider) === normalizar(usuario.vinculo);
  }
  return false;
}

function podeCriarProjeto(usuario: Usuario | null) {
  if (!usuario) return false;
  return usuario.cargo === "diretoria" || usuario.cargo === "lider";
}

function podeSubirVideoEditor(
  usuario: Usuario | null,
  projeto: Projeto | null
) {
  if (!usuario || !projeto) return false;
  if (usuario.cargo === "diretoria") return true;
  if (usuario.cargo === "editor") {
    return normalizar(projeto.Editor) === normalizar(usuario.vinculo);
  }
  return false;
}

function corStatus(status: string) {
  const s = normalizar(status);

  if (s.indexOf("conclu") !== -1 || s.indexOf("final") !== -1) {
    return { bg: "#e7f8ef", color: "#087443" };
  }
  if (s.indexOf("exec") !== -1 || s.indexOf("andamento") !== -1) {
    return { bg: "#eaf2ff", color: "#1456d9" };
  }
  if (s.indexOf("interromp") !== -1 || s.indexOf("paus") !== -1) {
    return { bg: "#fff1f0", color: "#c2410c" };
  }
  if (s.indexOf("stand") !== -1) {
    return { bg: "#eef4ff", color: "#3657c8" };
  }
  return { bg: "#f3f6fb", color: "#475467" };
}

function escaparCSV(valor?: string) {
  const texto = String(valor || "").replace(/"/g, '""');
  return `"${texto}"`;
}

function mapProjetoDb(item: any, elenco: ElencoItem[]): Projeto {
  return {
    ID: String(item.id),
    Projeto: item.projeto || "",
    Tipo: item.tipo || "",
    Genero: item.genero || "",
    Prioridade: item.prioridade || "",
    Dupla: item.dupla || "",
    Lider: item.lider || "",
    Telefone_Lider: item.telefone_lider || "",
    Editor: item.editor || "",
    Telefone_Editor: item.telefone_editor || "",
    Status: item.status || "",
    Data_Inicio: item.data_inicio || "",
    Video_Editor_Link: item.video_editor_link || "",
    Registro_Semanal: item.registro_semanal || "",
    Observacoes: item.observacoes || "",
    Elenco: elenco,
  };
}

async function carregarProjetosBanco(): Promise<Projeto[]> {
  const { data: projetosDb, error: erroProjetos } = await supabase
    .from("projetos")
    .select("*")
    .order("id", { ascending: false });

  if (erroProjetos) {
    console.error("Erro ao buscar projetos:", erroProjetos);
    return [];
  }

  const { data: elencoDb, error: erroElenco } = await supabase
    .from("elenco")
    .select("*")
    .order("id", { ascending: true });

  if (erroElenco) {
    console.error("Erro ao buscar elenco:", erroElenco);
    return [];
  }

  const elencoPorProjeto = new Map<string, ElencoItem[]>();

  (elencoDb || []).forEach((item: any) => {
    const projetoId = String(item.projeto_id);
    if (!elencoPorProjeto.has(projetoId)) {
      elencoPorProjeto.set(projetoId, []);
    }

    elencoPorProjeto.get(projetoId)!.push({
      id: String(item.id),
      personagem: item.personagem || "",
      dublador: item.dublador || "",
      funcao: item.funcao || "",
    });
  });

  return (projetosDb || []).map((item: any) =>
    mapProjetoDb(item, elencoPorProjeto.get(String(item.id)) || [])
  );
}

async function criarProjetoBanco(projeto: Projeto): Promise<string | null> {
  const { data, error } = await supabase
    .from("projetos")
    .insert([
      {
        projeto: projeto.Projeto,
        tipo: projeto.Tipo,
        genero: projeto.Genero,
        prioridade: projeto.Prioridade,
        dupla: projeto.Dupla,
        lider: projeto.Lider,
        telefone_lider: projeto.Telefone_Lider,
        editor: projeto.Editor,
        telefone_editor: projeto.Telefone_Editor,
        status: projeto.Status,
        data_inicio: projeto.Data_Inicio,
        video_editor_link: projeto.Video_Editor_Link,
        registro_semanal: projeto.Registro_Semanal,
        observacoes: projeto.Observacoes,
      },
    ])
    .select("id")
    .single();

  if (error) {
    console.error("Erro ao criar projeto:", error);
    return null;
  }

  return String(data.id);
}

async function atualizarProjetoBanco(projeto: Projeto): Promise<boolean> {
  const { error } = await supabase
    .from("projetos")
    .update({
      projeto: projeto.Projeto,
      tipo: projeto.Tipo,
      genero: projeto.Genero,
      prioridade: projeto.Prioridade,
      dupla: projeto.Dupla,
      lider: projeto.Lider,
      telefone_lider: projeto.Telefone_Lider,
      editor: projeto.Editor,
      telefone_editor: projeto.Telefone_Editor,
      status: projeto.Status,
      data_inicio: projeto.Data_Inicio,
      video_editor_link: projeto.Video_Editor_Link,
      registro_semanal: projeto.Registro_Semanal,
      observacoes: projeto.Observacoes,
    })
    .eq("id", Number(projeto.ID));

  if (error) {
    console.error("Erro ao atualizar projeto:", error);
    return false;
  }

  return true;
}

async function salvarElencoProjetoBanco(
  projetoId: string,
  elenco: ElencoItem[]
): Promise<boolean> {
  const { error: erroDelete } = await supabase
    .from("elenco")
    .delete()
    .eq("projeto_id", Number(projetoId));

  if (erroDelete) {
    console.error("Erro ao limpar elenco:", erroDelete);
    return false;
  }

  if (!elenco.length) return true;

  const payload = elenco.map((item) => ({
    projeto_id: Number(projetoId),
    personagem: item.personagem,
    dublador: item.dublador,
    funcao: item.funcao || "",
  }));

  const { error: erroInsert } = await supabase.from("elenco").insert(payload);

  if (erroInsert) {
    console.error("Erro ao salvar elenco:", erroInsert);
    return false;
  }

  return true;
}

function gerarIdTemporario() {
  return `${Date.now()}_${Math.floor(Math.random() * 100000)}`;
}

export default function App() {
  const [projetos, setProjetos] = useState<Projeto[]>([]);
  const [usuarios, setUsuarios] = useState<Usuario[]>([]);
  const [query, setQuery] = useState("");
  const [statusFiltro, setStatusFiltro] = useState("Todos");
  const [usuarioLogado, setUsuarioLogado] = useState<Usuario | null>(null);
  const [login, setLogin] = useState("");
  const [senha, setSenha] = useState("");
  const [erroLogin, setErroLogin] = useState("");
  const [selecionadoId, setSelecionadoId] = useState<string | null>(null);
  const [rascunho, setRascunho] = useState<Projeto | null>(null);
  const [mostrarNovoProjeto, setMostrarNovoProjeto] = useState(false);
  const [mostrarUsuarios, setMostrarUsuarios] = useState(false);
  const [carregando, setCarregando] = useState(true);

  const [novoProjeto, setNovoProjeto] = useState<Projeto>(projetoVazio);
  const [novoUsuario, setNovoUsuario] = useState<Usuario>({
    nome: "",
    login: "",
    senha: "",
    cargo: "lider",
    vinculo: "",
  });
  const [novoElencoProjeto, setNovoElencoProjeto] =
    useState<ElencoItem>(elencoVazio);
  const [novoElencoRascunho, setNovoElencoRascunho] =
    useState<ElencoItem>(elencoVazio);

  async function recarregarProjetos() {
    setCarregando(true);
    const lista = await carregarProjetosBanco();
    setProjetos(lista);
    setCarregando(false);
  }

  useEffect(() => {
    recarregarProjetos();

    const usuariosSalvos = carregarUsuariosLocal();
    if (usuariosSalvos.length) {
      setUsuarios(usuariosSalvos);
    } else {
      setUsuarios(usuariosPadrao);
      salvarUsuariosLocal(usuariosPadrao);
    }
  }, []);

  useEffect(() => {
    if (!mostrarNovoProjeto || !usuarioLogado) return;

    if (usuarioLogado.cargo === "lider") {
      setNovoProjeto((anterior) => ({
        ...anterior,
        Lider: usuarioLogado.vinculo || usuarioLogado.nome,
      }));
    }
  }, [mostrarNovoProjeto, usuarioLogado]);

  const projetosVisiveis = useMemo(() => {
    if (!usuarioLogado) return [];
    return projetos.filter((p) => podeVerProjeto(usuarioLogado, p));
  }, [projetos, usuarioLogado]);

  const statusUnicos = useMemo(() => {
    return Array.from(
      new Set(projetosVisiveis.map((p) => p.Status).filter(Boolean))
    );
  }, [projetosVisiveis]);

  const filtrados = useMemo(() => {
    return projetosVisiveis.filter((p) => {
      const termo = query.toLowerCase();

      const camposBusca = [
        p.ID,
        p.Projeto,
        p.Tipo,
        p.Genero,
        p.Prioridade,
        p.Status,
        p.Lider,
        p.Editor,
        p.Dupla,
        ...p.Elenco.map(
          (item) => `${item.personagem} ${item.dublador} ${item.funcao}`
        ),
      ]
        .join(" ")
        .toLowerCase();

      const busca = camposBusca.indexOf(termo) !== -1;
      const statusOk = statusFiltro === "Todos" || p.Status === statusFiltro;

      return busca && statusOk;
    });
  }, [projetosVisiveis, query, statusFiltro]);

  const selecionado = useMemo(() => {
    return projetos.find((p) => p.ID === selecionadoId) || null;
  }, [projetos, selecionadoId]);

  useEffect(() => {
    if (selecionado) {
      setRascunho({
        ...selecionado,
        Elenco: [...selecionado.Elenco],
      });
    } else {
      setRascunho(null);
    }
  }, [selecionado]);

  const rankingDubladores = useMemo(() => {
    const mapa = new Map<
      string,
      {
        nome: string;
        totalProjetos: number;
        totalPapeis: number;
        projetosUnicos: Set<string>;
      }
    >();

    projetos.forEach((projeto) => {
      projeto.Elenco.forEach((item) => {
        const chave = normalizar(item.dublador);
        if (!chave) return;

        if (!mapa.has(chave)) {
          mapa.set(chave, {
            nome: item.dublador,
            totalProjetos: 0,
            totalPapeis: 0,
            projetosUnicos: new Set<string>(),
          });
        }

        const registro = mapa.get(chave)!;
        registro.totalPapeis += 1;
        registro.projetosUnicos.add(projeto.ID);
      });
    });

    return Array.from(mapa.values())
      .map((item) => ({
        nome: item.nome,
        totalProjetos: item.projetosUnicos.size,
        totalPapeis: item.totalPapeis,
      }))
      .sort((a, b) => {
        if (b.totalProjetos !== a.totalProjetos)
          return b.totalProjetos - a.totalProjetos;
        return b.totalPapeis - a.totalPapeis;
      });
  }, [projetos]);

  const totalProjetos = projetosVisiveis.length;
  const totalAtivos = projetosVisiveis.filter(
    (p) =>
      ["finalizado", "concluida", "concluído"].indexOf(normalizar(p.Status)) ===
      -1
  ).length;
  const totalFinalizados = projetosVisiveis.filter(
    (p) =>
      ["finalizado", "concluida", "concluído"].indexOf(normalizar(p.Status)) !==
      -1
  ).length;
  const totalElenco = projetosVisiveis.reduce(
    (acc, projeto) => acc + projeto.Elenco.length,
    0
  );

  function fazerLogin(e: React.FormEvent) {
    e.preventDefault();

    const usuario = usuarios.find(
      (u) => u.login === login && u.senha === senha
    );

    if (!usuario) {
      setErroLogin("Login ou senha inválidos.");
      return;
    }

    setUsuarioLogado(usuario);
    setErroLogin("");
    setSelecionadoId(null);
  }

  function sair() {
    setUsuarioLogado(null);
    setLogin("");
    setSenha("");
    setErroLogin("");
    setSelecionadoId(null);
    setRascunho(null);
  }

  function atualizarCampo(campo: keyof Projeto, valor: string) {
    if (!rascunho) return;
    setRascunho({
      ...rascunho,
      [campo]: valor,
    });
  }

  function limparFormularioProjeto() {
    setNovoProjeto({
      ...projetoVazio,
      Lider:
        usuarioLogado && usuarioLogado.cargo === "lider"
          ? usuarioLogado.vinculo || usuarioLogado.nome
          : "",
    });
    setNovoElencoProjeto(elencoVazio);
  }

  async function criarProjeto() {
    if (!usuarioLogado || !podeCriarProjeto(usuarioLogado)) return;

    if (!novoProjeto.Projeto.trim()) {
      alert("Preencha pelo menos o nome do projeto.");
      return;
    }

    const projetoParaSalvar: Projeto = {
      ...novoProjeto,
      Lider:
        usuarioLogado.cargo === "lider"
          ? usuarioLogado.vinculo || usuarioLogado.nome
          : novoProjeto.Lider,
    };

    const projetoId = await criarProjetoBanco(projetoParaSalvar);

    if (!projetoId) {
      alert("Erro ao salvar no banco.");
      return;
    }

    const elencoOk = await salvarElencoProjetoBanco(
      projetoId,
      projetoParaSalvar.Elenco
    );

    if (!elencoOk) {
      alert("Projeto salvo, mas houve erro ao salvar o elenco.");
    }

    await recarregarProjetos();
    setMostrarNovoProjeto(false);
    limparFormularioProjeto();
    setSelecionadoId(projetoId);
    alert("Projeto salvo no banco 🚀");
  }

  async function salvarAlteracoes() {
    if (!rascunho || !selecionado || !usuarioLogado) return;

    const podeEditar = podeEditarProjeto(usuarioLogado, selecionado);
    const podeVideo = podeSubirVideoEditor(usuarioLogado, selecionado);

    if (!podeEditar && !podeVideo) return;

    if (!podeEditar && podeVideo) {
      const projetoVideo: Projeto = {
        ...selecionado,
        Video_Editor_Link: rascunho.Video_Editor_Link || "",
        Observacoes: rascunho.Observacoes || "",
      };

      const ok = await atualizarProjetoBanco(projetoVideo);
      if (!ok) {
        alert("Erro ao salvar no banco.");
        return;
      }

      await recarregarProjetos();
      alert("Alterações salvas com sucesso.");
      return;
    }

    const projetoFinal: Projeto = {
      ...rascunho,
      Lider:
        usuarioLogado.cargo === "lider"
          ? usuarioLogado.vinculo || usuarioLogado.nome
          : rascunho.Lider,
    };

    const projetoOk = await atualizarProjetoBanco(projetoFinal);
    if (!projetoOk) {
      alert("Erro ao salvar o projeto no banco.");
      return;
    }

    const elencoOk = await salvarElencoProjetoBanco(
      projetoFinal.ID,
      projetoFinal.Elenco
    );
    if (!elencoOk) {
      alert("Projeto salvo, mas houve erro ao salvar o elenco.");
      return;
    }

    await recarregarProjetos();
    alert("Alterações salvas com sucesso.");
  }

  function criarUsuario() {
    if (!novoUsuario.nome || !novoUsuario.login || !novoUsuario.senha) {
      alert("Preencha nome, login e senha.");
      return;
    }

    if (usuarios.some((u) => u.login === novoUsuario.login)) {
      alert("Já existe um usuário com esse login.");
      return;
    }

    const atualizados = [...usuarios, novoUsuario];
    setUsuarios(atualizados);
    salvarUsuariosLocal(atualizados);

    setNovoUsuario({
      nome: "",
      login: "",
      senha: "",
      cargo: "lider",
      vinculo: "",
    });

    alert("Usuário criado com sucesso.");
  }

  function excluirUsuario(loginUsuario: string) {
    if (loginUsuario === "diretoria") {
      alert("Não é possível excluir o usuário principal da diretoria.");
      return;
    }

    const atualizados = usuarios.filter((u) => u.login !== loginUsuario);
    setUsuarios(atualizados);
    salvarUsuariosLocal(atualizados);
  }

  function adicionarElencoNovoProjeto() {
    if (
      !novoElencoProjeto.personagem.trim() ||
      !novoElencoProjeto.dublador.trim()
    ) {
      alert("Preencha personagem e dublador.");
      return;
    }

    setNovoProjeto((anterior) => ({
      ...anterior,
      Elenco: [
        ...anterior.Elenco,
        {
          ...novoElencoProjeto,
          id: gerarIdTemporario(),
        },
      ],
    }));

    setNovoElencoProjeto(elencoVazio);
  }

  function removerElencoNovoProjeto(id: string) {
    setNovoProjeto((anterior) => ({
      ...anterior,
      Elenco: anterior.Elenco.filter((item) => item.id !== id),
    }));
  }

  function adicionarElencoRascunho() {
    if (!rascunho) return;

    if (
      !novoElencoRascunho.personagem.trim() ||
      !novoElencoRascunho.dublador.trim()
    ) {
      alert("Preencha personagem e dublador.");
      return;
    }

    setRascunho({
      ...rascunho,
      Elenco: [
        ...rascunho.Elenco,
        {
          ...novoElencoRascunho,
          id: gerarIdTemporario(),
        },
      ],
    });

    setNovoElencoRascunho(elencoVazio);
  }

  function removerElencoRascunho(id: string) {
    if (!rascunho) return;
    setRascunho({
      ...rascunho,
      Elenco: rascunho.Elenco.filter((item) => item.id !== id),
    });
  }

  function exportarProjetosCSV() {
    const cabecalhos = [
      "ID",
      "Projeto",
      "Tipo",
      "Genero",
      "Prioridade",
      "Dupla",
      "Lider",
      "Telefone_Lider",
      "Editor",
      "Telefone_Editor",
      "Status",
      "Data_Inicio",
      "Video_Editor_Link",
      "Registro_Semanal",
      "Observacoes",
      "Qtd_Elenco",
    ];

    const linhas = filtrados.map((p) =>
      [
        p.ID,
        p.Projeto,
        p.Tipo,
        p.Genero,
        p.Prioridade,
        p.Dupla,
        p.Lider,
        p.Telefone_Lider,
        p.Editor,
        p.Telefone_Editor,
        p.Status,
        p.Data_Inicio,
        p.Video_Editor_Link,
        p.Registro_Semanal,
        p.Observacoes,
        String(p.Elenco.length),
      ]
        .map(escaparCSV)
        .join(";")
    );

    const conteudo = "\uFEFF" + [cabecalhos.join(";"), ...linhas].join("\n");
    const blob = new Blob([conteudo], { type: "text/csv;charset=utf-8;" });
    const url = URL.createObjectURL(blob);
    const a = document.createElement("a");

    a.href = url;
    a.download = `projetos_dubworks_${new Date()
      .toISOString()
      .slice(0, 10)}.csv`;
    document.body.appendChild(a);
    a.click();
    document.body.removeChild(a);
    URL.revokeObjectURL(url);
  }

  function exportarElencoCSV() {
    const cabecalhos = [
      "Projeto",
      "ID_Projeto",
      "Personagem",
      "Dublador",
      "Funcao",
      "Qtd_Projetos_Dublador",
      "Qtd_Papeis_Dublador",
    ];

    const linhas: string[] = [];

    filtrados.forEach((projeto) => {
      projeto.Elenco.forEach((item) => {
        const ranking = rankingDubladores.find(
          (r) => normalizar(r.nome) === normalizar(item.dublador)
        );

        linhas.push(
          [
            projeto.Projeto,
            projeto.ID,
            item.personagem,
            item.dublador,
            item.funcao,
            String(ranking?.totalProjetos || 0),
            String(ranking?.totalPapeis || 0),
          ]
            .map(escaparCSV)
            .join(";")
        );
      });
    });

    const conteudo = "\uFEFF" + [cabecalhos.join(";"), ...linhas].join("\n");
    const blob = new Blob([conteudo], { type: "text/csv;charset=utf-8;" });
    const url = URL.createObjectURL(blob);
    const a = document.createElement("a");

    a.href = url;
    a.download = `elenco_dubworks_${new Date().toISOString().slice(0, 10)}.csv`;
    document.body.appendChild(a);
    a.click();
    document.body.removeChild(a);
    URL.revokeObjectURL(url);
  }

  if (!usuarioLogado) {
    return (
      <div
        style={{
          minHeight: "100vh",
          background:
            "linear-gradient(180deg, #0f3c8d 0%, #1456d9 45%, #f4f8fd 45%, #f4f8fd 100%)",
          display: "flex",
          justifyContent: "center",
          alignItems: "center",
          padding: 24,
          fontFamily: "Arial, sans-serif",
        }}
      >
        <div
          style={{
            width: "100%",
            maxWidth: 520,
            background: "#fff",
            borderRadius: 28,
            boxShadow: "0 30px 80px rgba(6, 31, 75, 0.22)",
            overflow: "hidden",
            border: "1px solid #dbe7f7",
          }}
        >
          <div
            style={{
              background: "linear-gradient(135deg, #0d47a1, #1366d9)",
              padding: 28,
              color: "#fff",
            }}
          >
            <img
              src={LOGO_URL}
              alt="DubWorks"
              style={{
                width: 150,
                height: "auto",
                display: "block",
                marginBottom: 14,
              }}
            />
            <h1 style={{ margin: 0, fontSize: 34 }}>DubWorks Manager</h1>
            <p style={{ marginTop: 8, opacity: 0.9, fontSize: 15 }}>
              Acesso interno de gerenciamento
            </p>
          </div>

          <form onSubmit={fazerLogin} style={{ padding: 28 }}>
            <label style={labelStyle}>Login</label>
            <input
              value={login}
              onChange={(e) => setLogin(e.target.value)}
              placeholder="Digite seu login"
              style={{ ...inputStyle, marginBottom: 16 }}
            />

            <label style={labelStyle}>Senha</label>
            <input
              type="password"
              value={senha}
              onChange={(e) => setSenha(e.target.value)}
              placeholder="Digite sua senha"
              style={{ ...inputStyle, marginBottom: 16 }}
            />

            {erroLogin && (
              <div
                style={{
                  background: "#fff2f0",
                  color: "#c2410c",
                  border: "1px solid #ffd7cc",
                  padding: 12,
                  borderRadius: 12,
                  marginBottom: 16,
                }}
              >
                {erroLogin}
              </div>
            )}

            <button type="submit" style={botaoPrimarioStyleGrande}>
              Entrar
            </button>
          </form>
        </div>
      </div>
    );
  }

  return (
    <div
      style={{
        minHeight: "100vh",
        background: estilos.fundo,
        fontFamily: "Arial, sans-serif",
        color: estilos.texto,
      }}
    >
      <header
        style={{
          background: estilos.branco,
          borderBottom: `1px solid ${estilos.borda}`,
          padding: "14px 24px",
          display: "flex",
          justifyContent: "space-between",
          alignItems: "center",
          gap: 20,
          flexWrap: "wrap",
          position: "sticky",
          top: 0,
          zIndex: 20,
        }}
      >
        <div style={{ display: "flex", alignItems: "center", gap: 18 }}>
          <img
            src={LOGO_URL}
            alt="DubWorks"
            style={{ width: 120, height: "auto", objectFit: "contain" }}
          />
          <div>
            <div
              style={{
                fontSize: 28,
                fontWeight: 800,
                color: estilos.azulEscuro,
              }}
            >
              DubWorks Manager
            </div>
            <div style={{ color: estilos.textoSuave, marginTop: 4 }}>
              Acesso interno de gerenciamento
            </div>
          </div>
        </div>

        <div
          style={{
            display: "flex",
            alignItems: "center",
            gap: 12,
            flexWrap: "wrap",
          }}
        >
          {usuarioLogado.cargo === "diretoria" && (
            <button
              onClick={() => setMostrarUsuarios(true)}
              style={botaoSecundarioStyle}
            >
              Usuários
            </button>
          )}

          {podeCriarProjeto(usuarioLogado) && (
            <button
              onClick={() => {
                limparFormularioProjeto();
                setMostrarNovoProjeto(true);
              }}
              style={botaoPrimarioStyle}
            >
              Novo projeto
            </button>
          )}

          <button onClick={exportarProjetosCSV} style={botaoSecundarioStyle}>
            Exportar projetos
          </button>

          <button onClick={exportarElencoCSV} style={botaoSecundarioStyle}>
            Exportar elenco
          </button>

          <button onClick={sair} style={botaoSecundarioStyle}>
            Sair
          </button>

          <div style={{ textAlign: "right", minWidth: 110 }}>
            <div style={{ fontWeight: 700 }}>{usuarioLogado.nome}</div>
            <div style={{ color: estilos.textoSuave, fontSize: 14 }}>
              {usuarioLogado.cargo === "diretoria"
                ? "Diretoria"
                : usuarioLogado.cargo === "lider"
                ? "Líder"
                : "Editor"}
            </div>
          </div>
        </div>
      </header>

      <main style={{ padding: 24 }}>
        <div
          style={{
            display: "grid",
            gridTemplateColumns: "repeat(auto-fit, minmax(220px, 1fr))",
            gap: 16,
            marginBottom: 18,
          }}
        >
          <ResumoCard
            titulo="Total de projetos"
            valor={String(totalProjetos)}
          />
          <ResumoCard titulo="Projetos ativos" valor={String(totalAtivos)} />
          <ResumoCard titulo="Finalizados" valor={String(totalFinalizados)} />
          <ResumoCard
            titulo="Registros de elenco"
            valor={String(totalElenco)}
          />
        </div>

        <div
          style={{
            ...cardStyle,
            marginBottom: 18,
            display: "grid",
            gridTemplateColumns: "1.4fr 220px 170px 170px",
            gap: 12,
            alignItems: "center",
          }}
        >
          <input
            placeholder="Buscar por ID, projeto, líder, editor, gênero, personagem..."
            value={query}
            onChange={(e) => setQuery(e.target.value)}
            style={inputStyle}
          />

          <select
            value={statusFiltro}
            onChange={(e) => setStatusFiltro(e.target.value)}
            style={inputStyle}
          >
            <option>Todos</option>
            {statusUnicos.map((status, i) => (
              <option key={i}>{status}</option>
            ))}
          </select>

          <button
            onClick={exportarProjetosCSV}
            style={botaoSecundarioGrandeStyle}
          >
            Relatório
          </button>

          <button
            onClick={recarregarProjetos}
            style={botaoSecundarioGrandeStyle}
          >
            Atualizar
          </button>
        </div>

        <div
          style={{
            display: "grid",
            gridTemplateColumns: "1.45fr 1fr",
            gap: 20,
            alignItems: "start",
          }}
        >
          <div style={{ ...cardStyle, padding: 0, overflow: "hidden" }}>
            <div
              style={{
                padding: "16px 18px",
                borderBottom: `1px solid ${estilos.borda}`,
                background: "linear-gradient(180deg, #f8fbff, #f1f6ff)",
                fontWeight: 800,
                color: estilos.azulEscuro,
                fontSize: 20,
              }}
            >
              Projetos {carregando ? "• carregando..." : ""}
            </div>

            <div style={{ overflowX: "auto" }}>
              <table
                style={{
                  width: "100%",
                  borderCollapse: "collapse",
                  minWidth: 980,
                }}
              >
                <thead>
                  <tr style={{ textAlign: "left", background: "#f8fbff" }}>
                    <th style={thStyle}>ID</th>
                    <th style={thStyle}>Projeto</th>
                    <th style={thStyle}>Tipo</th>
                    <th style={thStyle}>Gênero</th>
                    <th style={thStyle}>Prioridade</th>
                    <th style={thStyle}>Status</th>
                    <th style={thStyle}>Líder</th>
                    <th style={thStyle}>Editor</th>
                  </tr>
                </thead>
                <tbody>
                  {filtrados.map((p) => {
                    const ativo = p.ID === selecionadoId;
                    const statusCor = corStatus(p.Status);

                    return (
                      <tr
                        key={p.ID}
                        onClick={() => setSelecionadoId(p.ID)}
                        style={{
                          cursor: "pointer",
                          background: ativo ? "#eef5ff" : "#fff",
                          borderTop: `1px solid ${estilos.borda}`,
                        }}
                      >
                        <td style={tdStyle}>{p.ID}</td>
                        <td style={tdStyle}>
                          <div style={{ fontWeight: 700 }}>{p.Projeto}</div>
                          <div
                            style={{ color: estilos.textoSuave, fontSize: 13 }}
                          >
                            {p.Elenco.length} registro(s) de elenco
                          </div>
                        </td>
                        <td style={tdStyle}>{p.Tipo}</td>
                        <td style={tdStyle}>{p.Genero}</td>
                        <td style={tdStyle}>{p.Prioridade}</td>
                        <td style={tdStyle}>
                          <span
                            style={{
                              display: "inline-block",
                              padding: "7px 12px",
                              borderRadius: 999,
                              background: statusCor.bg,
                              color: statusCor.color,
                              fontWeight: 700,
                              fontSize: 13,
                            }}
                          >
                            {p.Status}
                          </span>
                        </td>
                        <td style={tdStyle}>{p.Lider}</td>
                        <td style={tdStyle}>{p.Editor}</td>
                      </tr>
                    );
                  })}
                </tbody>
              </table>
            </div>
          </div>

          <div style={{ display: "grid", gap: 18 }}>
            <div style={{ ...cardStyle, minHeight: 460 }}>
              {!rascunho ? (
                <div style={{ color: estilos.textoSuave, fontSize: 17 }}>
                  Selecione um projeto para ver e editar.
                </div>
              ) : (
                <>
                  <div
                    style={{
                      paddingBottom: 14,
                      marginBottom: 18,
                      borderBottom: `1px solid ${estilos.borda}`,
                    }}
                  >
                    <h2
                      style={{
                        margin: 0,
                        fontSize: 30,
                        color: estilos.azulEscuro,
                      }}
                    >
                      {rascunho.Projeto || "Projeto"}
                    </h2>
                    <div style={{ marginTop: 8, color: estilos.textoSuave }}>
                      ID: {rascunho.ID}
                    </div>
                    <div style={{ marginTop: 6 }}>
                      <strong>Líder:</strong> {rascunho.Lider || "-"}
                    </div>
                  </div>

                  <InfoPermissao
                    podeEditar={podeEditarProjeto(usuarioLogado, selecionado)}
                    podeVideo={podeSubirVideoEditor(usuarioLogado, selecionado)}
                  />

                  <div
                    style={{
                      display: "grid",
                      gridTemplateColumns: "1fr 1fr",
                      gap: 12,
                    }}
                  >
                    <Campo
                      label="Projeto"
                      value={rascunho.Projeto}
                      onChange={(v) => atualizarCampo("Projeto", v)}
                      disabled={!podeEditarProjeto(usuarioLogado, selecionado)}
                    />
                    <Campo
                      label="Tipo"
                      value={rascunho.Tipo}
                      onChange={(v) => atualizarCampo("Tipo", v)}
                      disabled={!podeEditarProjeto(usuarioLogado, selecionado)}
                    />
                    <Campo
                      label="Gênero"
                      value={rascunho.Genero}
                      onChange={(v) => atualizarCampo("Genero", v)}
                      disabled={!podeEditarProjeto(usuarioLogado, selecionado)}
                    />
                    <Campo
                      label="Prioridade"
                      value={rascunho.Prioridade}
                      onChange={(v) => atualizarCampo("Prioridade", v)}
                      disabled={!podeEditarProjeto(usuarioLogado, selecionado)}
                    />
                    <Campo
                      label="Dupla"
                      value={rascunho.Dupla}
                      onChange={(v) => atualizarCampo("Dupla", v)}
                      disabled={!podeEditarProjeto(usuarioLogado, selecionado)}
                    />
                    <Campo
                      label="Status"
                      value={rascunho.Status}
                      onChange={(v) => atualizarCampo("Status", v)}
                      disabled={!podeEditarProjeto(usuarioLogado, selecionado)}
                    />
                    <Campo
                      label="Líder"
                      value={rascunho.Lider}
                      onChange={(v) => atualizarCampo("Lider", v)}
                      disabled={usuarioLogado.cargo !== "diretoria"}
                    />
                    <Campo
                      label="Telefone do líder"
                      value={rascunho.Telefone_Lider}
                      onChange={(v) => atualizarCampo("Telefone_Lider", v)}
                      disabled={!podeEditarProjeto(usuarioLogado, selecionado)}
                    />
                    <Campo
                      label="Editor"
                      value={rascunho.Editor}
                      onChange={(v) => atualizarCampo("Editor", v)}
                      disabled={usuarioLogado.cargo !== "diretoria"}
                    />
                    <Campo
                      label="Telefone do editor"
                      value={rascunho.Telefone_Editor}
                      onChange={(v) => atualizarCampo("Telefone_Editor", v)}
                      disabled={!podeEditarProjeto(usuarioLogado, selecionado)}
                    />
                    <Campo
                      label="Data de início"
                      value={rascunho.Data_Inicio}
                      onChange={(v) => atualizarCampo("Data_Inicio", v)}
                      disabled={!podeEditarProjeto(usuarioLogado, selecionado)}
                    />
                  </div>

                  <div style={{ marginTop: 14 }}>
                    <label style={labelStyle}>Registro semanal do líder</label>
                    <textarea
                      value={rascunho.Registro_Semanal}
                      onChange={(e) =>
                        atualizarCampo("Registro_Semanal", e.target.value)
                      }
                      disabled={!podeEditarProjeto(usuarioLogado, selecionado)}
                      rows={4}
                      style={{
                        ...textareaStyle,
                        background: podeEditarProjeto(
                          usuarioLogado,
                          selecionado
                        )
                          ? "#fff"
                          : "#f4f6f8",
                      }}
                    />
                  </div>

                  <div style={{ marginTop: 14 }}>
                    <label style={labelStyle}>Link do vídeo do editor</label>
                    <input
                      value={rascunho.Video_Editor_Link}
                      onChange={(e) =>
                        atualizarCampo("Video_Editor_Link", e.target.value)
                      }
                      disabled={
                        !podeSubirVideoEditor(usuarioLogado, selecionado)
                      }
                      style={{
                        ...inputStyle,
                        background: podeSubirVideoEditor(
                          usuarioLogado,
                          selecionado
                        )
                          ? "#fff"
                          : "#f4f6f8",
                      }}
                    />
                  </div>

                  <div style={{ marginTop: 14 }}>
                    <label style={labelStyle}>Observações</label>
                    <textarea
                      value={rascunho.Observacoes}
                      onChange={(e) =>
                        atualizarCampo("Observacoes", e.target.value)
                      }
                      disabled={
                        !podeEditarProjeto(usuarioLogado, selecionado) &&
                        !podeSubirVideoEditor(usuarioLogado, selecionado)
                      }
                      rows={4}
                      style={{
                        ...textareaStyle,
                        background:
                          podeEditarProjeto(usuarioLogado, selecionado) ||
                          podeSubirVideoEditor(usuarioLogado, selecionado)
                            ? "#fff"
                            : "#f4f6f8",
                      }}
                    />
                  </div>

                  <button
                    onClick={salvarAlteracoes}
                    disabled={
                      !podeEditarProjeto(usuarioLogado, selecionado) &&
                      !podeSubirVideoEditor(usuarioLogado, selecionado)
                    }
                    style={{
                      ...botaoPrimarioStyleGrande,
                      marginTop: 16,
                      opacity:
                        !podeEditarProjeto(usuarioLogado, selecionado) &&
                        !podeSubirVideoEditor(usuarioLogado, selecionado)
                          ? 0.6
                          : 1,
                    }}
                  >
                    Salvar alterações
                  </button>
                </>
              )}
            </div>

            <div style={cardStyle}>
              <div style={secaoTituloStyle}>Elenco do projeto</div>

              {!rascunho ? (
                <div style={{ color: estilos.textoSuave }}>
                  Selecione um projeto.
                </div>
              ) : (
                <>
                  {podeEditarProjeto(usuarioLogado, selecionado) && (
                    <div
                      style={{
                        display: "grid",
                        gridTemplateColumns: "1fr 1fr 1fr auto",
                        gap: 10,
                        marginBottom: 14,
                      }}
                    >
                      <input
                        placeholder="Personagem"
                        value={novoElencoRascunho.personagem}
                        onChange={(e) =>
                          setNovoElencoRascunho({
                            ...novoElencoRascunho,
                            personagem: e.target.value,
                          })
                        }
                        style={inputStyle}
                      />
                      <input
                        placeholder="Dublador"
                        value={novoElencoRascunho.dublador}
                        onChange={(e) =>
                          setNovoElencoRascunho({
                            ...novoElencoRascunho,
                            dublador: e.target.value,
                          })
                        }
                        style={inputStyle}
                      />
                      <input
                        placeholder="Função"
                        value={novoElencoRascunho.funcao}
                        onChange={(e) =>
                          setNovoElencoRascunho({
                            ...novoElencoRascunho,
                            funcao: e.target.value,
                          })
                        }
                        style={inputStyle}
                      />
                      <button
                        onClick={adicionarElencoRascunho}
                        style={botaoPrimarioStyle}
                      >
                        Adicionar
                      </button>
                    </div>
                  )}

                  {rascunho.Elenco.length === 0 ? (
                    <div style={{ color: estilos.textoSuave }}>
                      Nenhum personagem registrado ainda.
                    </div>
                  ) : (
                    <div style={{ display: "grid", gap: 10 }}>
                      {rascunho.Elenco.map((item) => {
                        const ranking = rankingDubladores.find(
                          (r) =>
                            normalizar(r.nome) === normalizar(item.dublador)
                        );

                        return (
                          <div
                            key={item.id}
                            style={{
                              border: `1px solid ${estilos.borda}`,
                              borderRadius: 16,
                              padding: 14,
                              background: "#fbfdff",
                              display: "flex",
                              justifyContent: "space-between",
                              gap: 14,
                              alignItems: "center",
                            }}
                          >
                            <div>
                              <div style={{ fontWeight: 800 }}>
                                {item.personagem}
                              </div>
                              <div style={{ marginTop: 4 }}>
                                <strong>Dublador:</strong> {item.dublador}
                              </div>
                              <div
                                style={{
                                  color: estilos.textoSuave,
                                  marginTop: 4,
                                }}
                              >
                                {item.funcao || "Sem função informada"}
                              </div>
                              <div
                                style={{
                                  marginTop: 8,
                                  display: "inline-block",
                                  background: "#eaf2ff",
                                  color: estilos.azulEscuro,
                                  padding: "6px 10px",
                                  borderRadius: 999,
                                  fontWeight: 700,
                                  fontSize: 12,
                                }}
                              >
                                {ranking?.totalProjetos || 0} projeto(s) •{" "}
                                {ranking?.totalPapeis || 0} papel(is)
                              </div>
                            </div>

                            {podeEditarProjeto(usuarioLogado, selecionado) && (
                              <button
                                onClick={() => removerElencoRascunho(item.id)}
                                style={botaoSecundarioStyle}
                              >
                                Remover
                              </button>
                            )}
                          </div>
                        );
                      })}
                    </div>
                  )}
                </>
              )}
            </div>

            <div style={cardStyle}>
              <div style={secaoTituloStyle}>Top dubladores</div>

              {rankingDubladores.length === 0 ? (
                <div style={{ color: estilos.textoSuave }}>
                  Ainda não há registros suficientes.
                </div>
              ) : (
                <div style={{ display: "grid", gap: 10 }}>
                  {rankingDubladores.slice(0, 8).map((item, idx) => (
                    <div
                      key={item.nome + idx}
                      style={{
                        display: "flex",
                        justifyContent: "space-between",
                        alignItems: "center",
                        padding: "12px 14px",
                        borderRadius: 14,
                        background: idx === 0 ? "#edf4ff" : "#f8fbff",
                        border: `1px solid ${estilos.borda}`,
                      }}
                    >
                      <div style={{ fontWeight: 700 }}>
                        {idx + 1}. {item.nome}
                      </div>
                      <div
                        style={{
                          background: estilos.azul,
                          color: "#fff",
                          borderRadius: 999,
                          padding: "6px 10px",
                          fontWeight: 700,
                          fontSize: 13,
                        }}
                      >
                        {item.totalProjetos} proj. / {item.totalPapeis} papéis
                      </div>
                    </div>
                  ))}
                </div>
              )}
            </div>
          </div>
        </div>
      </main>

      {mostrarNovoProjeto && (
        <Modal
          titulo="Novo projeto"
          onClose={() => setMostrarNovoProjeto(false)}
        >
          <div
            style={{
              display: "grid",
              gridTemplateColumns: "1fr 1fr",
              gap: 12,
            }}
          >
            <Campo
              label="Projeto"
              value={novoProjeto.Projeto}
              onChange={(v) => setNovoProjeto({ ...novoProjeto, Projeto: v })}
            />
            <Campo
              label="Tipo"
              value={novoProjeto.Tipo}
              onChange={(v) => setNovoProjeto({ ...novoProjeto, Tipo: v })}
            />
            <Campo
              label="Gênero"
              value={novoProjeto.Genero}
              onChange={(v) => setNovoProjeto({ ...novoProjeto, Genero: v })}
            />
            <Campo
              label="Prioridade"
              value={novoProjeto.Prioridade}
              onChange={(v) =>
                setNovoProjeto({ ...novoProjeto, Prioridade: v })
              }
            />
            <Campo
              label="Dupla"
              value={novoProjeto.Dupla}
              onChange={(v) => setNovoProjeto({ ...novoProjeto, Dupla: v })}
            />
            <Campo
              label="Líder"
              value={novoProjeto.Lider}
              onChange={(v) => setNovoProjeto({ ...novoProjeto, Lider: v })}
              disabled={usuarioLogado.cargo === "lider"}
            />
            <Campo
              label="Telefone do líder"
              value={novoProjeto.Telefone_Lider}
              onChange={(v) =>
                setNovoProjeto({ ...novoProjeto, Telefone_Lider: v })
              }
            />
            <Campo
              label="Editor"
              value={novoProjeto.Editor}
              onChange={(v) => setNovoProjeto({ ...novoProjeto, Editor: v })}
            />
            <Campo
              label="Telefone do editor"
              value={novoProjeto.Telefone_Editor}
              onChange={(v) =>
                setNovoProjeto({ ...novoProjeto, Telefone_Editor: v })
              }
            />
            <Campo
              label="Status"
              value={novoProjeto.Status}
              onChange={(v) => setNovoProjeto({ ...novoProjeto, Status: v })}
            />
            <Campo
              label="Data de início"
              value={novoProjeto.Data_Inicio}
              onChange={(v) =>
                setNovoProjeto({ ...novoProjeto, Data_Inicio: v })
              }
            />
          </div>

          <div style={{ marginTop: 16 }}>
            <label style={labelStyle}>Observações</label>
            <textarea
              value={novoProjeto.Observacoes}
              onChange={(e) =>
                setNovoProjeto({ ...novoProjeto, Observacoes: e.target.value })
              }
              rows={3}
              style={textareaStyle}
            />
          </div>

          <div style={{ marginTop: 18 }}>
            <div style={secaoTituloStyle}>Elenco inicial do projeto</div>

            <div
              style={{
                display: "grid",
                gridTemplateColumns: "1fr 1fr 1fr auto",
                gap: 10,
                marginBottom: 12,
              }}
            >
              <input
                placeholder="Personagem"
                value={novoElencoProjeto.personagem}
                onChange={(e) =>
                  setNovoElencoProjeto({
                    ...novoElencoProjeto,
                    personagem: e.target.value,
                  })
                }
                style={inputStyle}
              />
              <input
                placeholder="Dublador"
                value={novoElencoProjeto.dublador}
                onChange={(e) =>
                  setNovoElencoProjeto({
                    ...novoElencoProjeto,
                    dublador: e.target.value,
                  })
                }
                style={inputStyle}
              />
              <input
                placeholder="Função"
                value={novoElencoProjeto.funcao}
                onChange={(e) =>
                  setNovoElencoProjeto({
                    ...novoElencoProjeto,
                    funcao: e.target.value,
                  })
                }
                style={inputStyle}
              />
              <button
                onClick={adicionarElencoNovoProjeto}
                style={botaoPrimarioStyle}
              >
                Adicionar
              </button>
            </div>

            {novoProjeto.Elenco.length > 0 && (
              <div style={{ display: "grid", gap: 8 }}>
                {novoProjeto.Elenco.map((item) => (
                  <div
                    key={item.id}
                    style={{
                      display: "flex",
                      justifyContent: "space-between",
                      alignItems: "center",
                      padding: "12px 14px",
                      borderRadius: 14,
                      background: "#f8fbff",
                      border: `1px solid ${estilos.borda}`,
                    }}
                  >
                    <div>
                      <strong>{item.personagem}</strong> — {item.dublador}
                      <div style={{ color: estilos.textoSuave, marginTop: 4 }}>
                        {item.funcao || "Sem função informada"}
                      </div>
                    </div>

                    <button
                      onClick={() => removerElencoNovoProjeto(item.id)}
                      style={botaoSecundarioStyle}
                    >
                      Remover
                    </button>
                  </div>
                ))}
              </div>
            )}
          </div>

          <button
            onClick={criarProjeto}
            style={{ ...botaoPrimarioStyleGrande, marginTop: 20 }}
          >
            Criar projeto
          </button>
        </Modal>
      )}

      {mostrarUsuarios && (
        <Modal
          titulo="Gestão de usuários"
          onClose={() => setMostrarUsuarios(false)}
          largo
        >
          <div style={{ ...cardStyle, boxShadow: "none", marginBottom: 16 }}>
            <div style={secaoTituloStyle}>Novo usuário</div>

            <div
              style={{
                display: "grid",
                gridTemplateColumns: "1fr 1fr",
                gap: 12,
              }}
            >
              <Campo
                label="Nome"
                value={novoUsuario.nome}
                onChange={(v) => setNovoUsuario({ ...novoUsuario, nome: v })}
              />
              <Campo
                label="Login"
                value={novoUsuario.login}
                onChange={(v) => setNovoUsuario({ ...novoUsuario, login: v })}
              />
              <Campo
                label="Senha"
                value={novoUsuario.senha}
                onChange={(v) => setNovoUsuario({ ...novoUsuario, senha: v })}
              />

              <div>
                <label style={labelStyle}>Cargo</label>
                <select
                  value={novoUsuario.cargo}
                  onChange={(e) =>
                    setNovoUsuario({
                      ...novoUsuario,
                      cargo: e.target.value as Cargo,
                    })
                  }
                  style={inputStyle}
                >
                  <option value="diretoria">Diretoria</option>
                  <option value="lider">Líder</option>
                  <option value="editor">Editor</option>
                </select>
              </div>

              <div style={{ gridColumn: "1 / -1" }}>
                <Campo
                  label="Vínculo de acesso"
                  value={novoUsuario.vinculo}
                  onChange={(v) =>
                    setNovoUsuario({ ...novoUsuario, vinculo: v })
                  }
                />
                <p
                  style={{
                    marginTop: 8,
                    color: estilos.textoSuave,
                    fontSize: 13,
                  }}
                >
                  Para líder, use exatamente o valor do campo{" "}
                  <strong>Lider</strong>. Para editor, use exatamente o valor do
                  campo <strong>Editor</strong>. Para diretoria, use{" "}
                  <strong>todos</strong>.
                </p>
              </div>
            </div>

            <button
              onClick={criarUsuario}
              style={{ ...botaoPrimarioStyleGrande, marginTop: 16 }}
            >
              Criar usuário
            </button>
          </div>

          <div style={{ ...cardStyle, boxShadow: "none", padding: 0 }}>
            <div style={{ overflowX: "auto" }}>
              <table style={{ width: "100%", borderCollapse: "collapse" }}>
                <thead>
                  <tr style={{ background: "#f8fbff", textAlign: "left" }}>
                    <th style={thStyle}>Nome</th>
                    <th style={thStyle}>Login</th>
                    <th style={thStyle}>Cargo</th>
                    <th style={thStyle}>Vínculo</th>
                    <th style={thStyle}>Ações</th>
                  </tr>
                </thead>
                <tbody>
                  {usuarios.map((u) => (
                    <tr
                      key={u.login}
                      style={{ borderTop: `1px solid ${estilos.borda}` }}
                    >
                      <td style={tdStyle}>{u.nome}</td>
                      <td style={tdStyle}>{u.login}</td>
                      <td style={tdStyle}>{u.cargo}</td>
                      <td style={tdStyle}>{u.vinculo}</td>
                      <td style={tdStyle}>
                        <button
                          onClick={() => excluirUsuario(u.login)}
                          style={botaoSecundarioStyle}
                        >
                          Excluir
                        </button>
                      </td>
                    </tr>
                  ))}
                </tbody>
              </table>
            </div>
          </div>
        </Modal>
      )}
    </div>
  );
}

function Campo({
  label,
  value,
  onChange,
  disabled,
}: {
  label: string;
  value: string;
  onChange: (valor: string) => void;
  disabled?: boolean;
}) {
  return (
    <div>
      <label style={labelStyle}>{label}</label>
      <input
        value={value}
        onChange={(e) => onChange(e.target.value)}
        disabled={disabled}
        style={{
          ...inputStyle,
          background: disabled ? "#f4f6f8" : "#fff",
        }}
      />
    </div>
  );
}

function Modal({
  titulo,
  onClose,
  children,
  largo,
}: {
  titulo: string;
  onClose: () => void;
  children: React.ReactNode;
  largo?: boolean;
}) {
  return (
    <div style={overlayStyle}>
      <div
        style={{
          width: "100%",
          maxWidth: largo ? 980 : 860,
          background: "#fff",
          borderRadius: 24,
          padding: 24,
          boxShadow: "0 30px 80px rgba(11, 61, 145, 0.25)",
          border: `1px solid ${estilos.borda}`,
          maxHeight: "90vh",
          overflowY: "auto",
        }}
      >
        <div
          style={{
            display: "flex",
            justifyContent: "space-between",
            alignItems: "center",
            marginBottom: 18,
          }}
        >
          <h2 style={{ margin: 0, fontSize: 30, color: estilos.azulEscuro }}>
            {titulo}
          </h2>
          <button onClick={onClose} style={botaoSecundarioStyle}>
            Fechar
          </button>
        </div>
        {children}
      </div>
    </div>
  );
}

function ResumoCard({ titulo, valor }: { titulo: string; valor: string }) {
  return (
    <div
      style={{
        ...cardStyle,
        background: "linear-gradient(180deg, #ffffff, #f8fbff)",
      }}
    >
      <div
        style={{ color: estilos.textoSuave, fontSize: 15, marginBottom: 12 }}
      >
        {titulo}
      </div>
      <div style={{ fontSize: 40, fontWeight: 800, color: estilos.azulEscuro }}>
        {valor}
      </div>
    </div>
  );
}

function InfoPermissao({
  podeEditar,
  podeVideo,
}: {
  podeEditar: boolean;
  podeVideo: boolean;
}) {
  let texto = "Você só pode visualizar este projeto.";

  if (podeEditar) texto = "Você pode editar este projeto.";
  if (!podeEditar && podeVideo) {
    texto =
      "Você pode visualizar este projeto e subir somente o vídeo do editor.";
  }

  return (
    <div
      style={{
        marginBottom: 16,
        padding: 12,
        borderRadius: 14,
        background: "#f6faff",
        border: `1px solid ${estilos.borda}`,
        color: estilos.texto,
        fontSize: 14,
        lineHeight: 1.6,
        fontWeight: 700,
      }}
    >
      {texto}
    </div>
  );
}

const cardStyle: React.CSSProperties = {
  background: "#fff",
  border: `1px solid ${estilos.borda}`,
  borderRadius: 22,
  padding: 18,
  boxShadow: "0 10px 28px rgba(17, 74, 156, 0.06)",
};

const secaoTituloStyle: React.CSSProperties = {
  fontSize: 22,
  fontWeight: 800,
  color: estilos.azulEscuro,
  marginBottom: 14,
};

const thStyle: React.CSSProperties = {
  padding: "15px 14px",
  fontSize: 14,
  color: estilos.azulEscuro,
  borderBottom: `1px solid ${estilos.borda}`,
};

const tdStyle: React.CSSProperties = {
  padding: "15px 14px",
  fontSize: 14,
  verticalAlign: "top",
};

const labelStyle: React.CSSProperties = {
  display: "block",
  marginBottom: 7,
  fontSize: 13,
  fontWeight: 700,
  color: estilos.texto,
};

const inputStyle: React.CSSProperties = {
  width: "100%",
  boxSizing: "border-box",
  padding: 13,
  borderRadius: 14,
  border: `1px solid ${estilos.borda}`,
  fontSize: 15,
  outline: "none",
};

const textareaStyle: React.CSSProperties = {
  width: "100%",
  boxSizing: "border-box",
  padding: 13,
  borderRadius: 14,
  border: `1px solid ${estilos.borda}`,
  fontSize: 15,
  resize: "vertical",
  outline: "none",
};

const overlayStyle: React.CSSProperties = {
  position: "fixed",
  inset: 0,
  background: "rgba(8, 33, 79, 0.45)",
  display: "flex",
  alignItems: "center",
  justifyContent: "center",
  padding: 24,
  zIndex: 40,
};

const botaoPrimarioStyle: React.CSSProperties = {
  background: "linear-gradient(135deg, #0d47a1, #1366d9)",
  color: "#fff",
  border: "none",
  borderRadius: 14,
  padding: "11px 16px",
  cursor: "pointer",
  fontWeight: 700,
  boxShadow: "0 8px 20px rgba(19, 102, 217, 0.22)",
};

const botaoPrimarioStyleGrande: React.CSSProperties = {
  width: "100%",
  background: "linear-gradient(135deg, #0d47a1, #1366d9)",
  color: "#fff",
  border: "none",
  borderRadius: 16,
  padding: 16,
  fontSize: 16,
  fontWeight: 700,
  cursor: "pointer",
  boxShadow: "0 10px 24px rgba(19, 102, 217, 0.22)",
};

const botaoSecundarioStyle: React.CSSProperties = {
  background: "#fff",
  border: `1px solid ${estilos.borda}`,
  borderRadius: 14,
  padding: "11px 15px",
  cursor: "pointer",
  fontWeight: 700,
  color: estilos.azulEscuro,
};

const botaoSecundarioGrandeStyle: React.CSSProperties = {
  background: "#f8fbff",
  border: `1px solid ${estilos.borda}`,
  borderRadius: 14,
  padding: 14,
  cursor: "pointer",
  fontWeight: 700,
  color: estilos.azulEscuro,
};
