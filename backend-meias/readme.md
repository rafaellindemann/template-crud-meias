rotas disponíveis (resumo):
POST /usuarios — cadastrar divulgador.
POST /auth/login — login simples (retorna {id,nome,email}).
GET /produtos?q= — listar (ordem alfabética; busca opcional).
GET /produtos/:id — obter 1.
POST /produtos — criar.
PUT /produtos/:id — atualizar.
DELETE /produtos/:id — excluir.
POST /movimentacoes — registrar entrada/saída (transação: atualiza saldo e grava histórico).
GET /movimentacoes?produto_id= — histórico geral ou por produto.
GET /health — ping.



-- Banco: saep_db
-- Script simplificado e compatível com a prova “meia meia meia”

-- ==========================
-- Tabelas
-- ==========================
CREATE TABLE IF NOT EXISTS usuarios (
  id     SERIAL PRIMARY KEY,
  nome   TEXT NOT NULL,
  email  TEXT NOT NULL UNIQUE,
  senha  TEXT NOT NULL
);

CREATE TABLE IF NOT EXISTS produtos (
  id              SERIAL PRIMARY KEY,
  nome            TEXT NOT NULL,
  quantidade      INTEGER NOT NULL DEFAULT 0,
  estoque_minimo  INTEGER NOT NULL DEFAULT 0
);

CREATE TABLE IF NOT EXISTS movimentacoes (
  id                 SERIAL PRIMARY KEY,
  produto_id         INTEGER NOT NULL REFERENCES produtos(id),
  usuario_id         INTEGER NOT NULL REFERENCES usuarios(id),
  tipo               TEXT NOT NULL,             -- 'entrada' | 'saida'
  quantidade         INTEGER NOT NULL,
  data_movimentacao  TIMESTAMP NOT NULL DEFAULT NOW(),
  observacao         TEXT
);

-- ==========================
-- SEEDS (≥3 por tabela)
-- ==========================

-- Usuários (divulgadores)
INSERT INTO usuarios (nome, email, senha) VALUES
  ('Ana Souza',  'ana@example.com',   '123'),
  ('Bruno Lima', 'bruno@example.com', '123'),
  ('Carla Dias', 'carla@example.com', '123')
ON CONFLICT (email) DO NOTHING;

-- Produtos (modelos oficiais da "meia meia meia")
INSERT INTO produtos (nome, quantidade, estoque_minimo) VALUES
  ('meia meia meia arrastão', 40, 10),
  ('499,5 (meia meia meia 3/4)', 60, 15),
  ('000 (meia meia meia de cano invisível)', 25, 12)
ON CONFLICT DO NOTHING;

-- Movimentações históricas (exemplo inicial de uso do sistema)
-- Entradas iniciais (Ana)
INSERT INTO movimentacoes (produto_id, usuario_id, tipo, quantidade, data_movimentacao, observacao) VALUES
  ( (SELECT id FROM produtos WHERE nome='meia meia meia arrastão'),
    (SELECT id FROM usuarios WHERE email='ana@example.com'),
    'entrada', 30, NOW() - INTERVAL '2 days', 'Compra inicial' ),
  ( (SELECT id FROM produtos WHERE nome='499,5 (meia meia meia 3/4)'),
    (SELECT id FROM usuarios WHERE email='ana@example.com'),
    'entrada', 50, NOW() - INTERVAL '2 days', 'Compra inicial' ),
  ( (SELECT id FROM produtos WHERE nome='000 (meia meia meia de cano invisível)'),
    (SELECT id FROM usuarios WHERE email='ana@example.com'),
    'entrada', 20, NOW() - INTERVAL '2 days', 'Compra inicial' );

-- Saídas (Bruno)
INSERT INTO movimentacoes (produto_id, usuario_id, tipo, quantidade, data_movimentacao, observacao) VALUES
  ( (SELECT id FROM produtos WHERE nome='meia meia meia arrastão'),
    (SELECT id FROM usuarios WHERE email='bruno@example.com'),
    'saida', 6, NOW() - INTERVAL '1 day', 'Retirada para evento' ),
  ( (SELECT id FROM produtos WHERE nome='499,5 (meia meia meia 3/4)'),
    (SELECT id FROM usuarios WHERE email='bruno@example.com'),
    'saida', 15, NOW() - INTERVAL '1 day', 'Retirada para feira' ),
  ( (SELECT id FROM produtos WHERE nome='000 (meia meia meia de cano invisível)'),
    (SELECT id FROM usuarios WHERE email='bruno@example.com'),
    'saida', 4, NOW() - INTERVAL '1 day', 'Retirada para divulgação' );

-- Reposição (Carla)
INSERT INTO movimentacoes (produto_id, usuario_id, tipo, quantidade, observacao) VALUES
  ( (SELECT id FROM produtos WHERE nome='meia meia meia arrastão'),
    (SELECT id FROM usuarios WHERE email='carla@example.com'),
    'entrada', 10, 'Devolução de kits' ),
  ( (SELECT id FROM produtos WHERE nome='499,5 (meia meia meia 3/4)'),
    (SELECT id FROM usuarios WHERE email='carla@example.com'),
    'entrada', 20, 'Devolução de kits' ),
  ( (SELECT id FROM produtos WHERE nome='000 (meia meia meia de cano invisível)'),
    (SELECT id FROM usuarios WHERE email='carla@example.com'),
    'entrada', 8, 'Devolução de kits' );
