# 💰 MÓDULO 5 — FINANCEIRO (ERP-LOJA)

📂 Estrutura de pastas no backend:

backend/
├── models/
│   └── financeiroModel.js
├── controllers/
│   └── financeiroController.js
├── routes/
│   └── financeiroRoutes.js
└── server.js

🔹 1. Banco de Dados (MySQL)

Adicione as tabelas no seu banco erp_db:

-- Tabela de Contas a Receber
CREATE TABLE IF NOT EXISTS contas_receber (
  id INT AUTO_INCREMENT PRIMARY KEY,
  cliente_id INT,
  descricao VARCHAR(200),
  valor DECIMAL(10,2),
  data_vencimento DATE,
  status ENUM('Pendente', 'Pago') DEFAULT 'Pendente',
  FOREIGN KEY (cliente_id) REFERENCES clientes(id)
);

-- Tabela de Contas a Pagar
CREATE TABLE IF NOT EXISTS contas_pagar (
  id INT AUTO_INCREMENT PRIMARY KEY,
  fornecedor VARCHAR(100),
  descricao VARCHAR(200),
  valor DECIMAL(10,2),
  data_vencimento DATE,
  status ENUM('Pendente', 'Pago') DEFAULT 'Pendente'
);

-- Tabela de Movimentações (Fluxo de Caixa)
CREATE TABLE IF NOT EXISTS fluxo_caixa (
  id INT AUTO_INCREMENT PRIMARY KEY,
  tipo ENUM('Entrada', 'Saída') NOT NULL,
  descricao VARCHAR(200),
  valor DECIMAL(10,2),
  data_movimento DATE DEFAULT CURRENT_DATE
);

🔹 2. models/financeiroModel.js
import db from "../config/db.js";

// Contas a Receber
export const getContasReceber = async () => {
  const [rows] = await db.query(`
    SELECT cr.*, c.nome AS cliente
    FROM contas_receber cr
    LEFT JOIN clientes c ON cr.cliente_id = c.id
  `);
  return rows;
};

// Contas a Pagar
export const getContasPagar = async () => {
  const [rows] = await db.query("SELECT * FROM contas_pagar");
  return rows;
};

// Fluxo de Caixa
export const getFluxoCaixa = async () => {
  const [rows] = await db.query("SELECT * FROM fluxo_caixa ORDER BY data_movimento DESC");
  return rows;
};

// Inserção de movimentações automáticas
export const addMovimento = async (tipo, descricao, valor) => {
  await db.query(
    "INSERT INTO fluxo_caixa (tipo, descricao, valor) VALUES (?, ?, ?)",
    [tipo, descricao, valor]
  );
};

🔹 3. controllers/financeiroController.js
import * as Fin from "../models/financeiroModel.js";

// Listar Contas
export const listarContasReceber = async (req, res) => {
  try {
    const data = await Fin.getContasReceber();
    res.json(data);
  } catch (err) {
    res.status(500).json({ message: "Erro ao buscar contas a receber", err });
  }
};

export const listarContasPagar = async (req, res) => {
  try {
    const data = await Fin.getContasPagar();
    res.json(data);
  } catch (err) {
    res.status(500).json({ message: "Erro ao buscar contas a pagar", err });
  }
};

export const listarFluxo = async (req, res) => {
  try {
    const data = await Fin.getFluxoCaixa();
    res.json(data);
  } catch (err) {
    res.status(500).json({ message: "Erro ao buscar fluxo de caixa", err });
  }
};

🔹 4. routes/financeiroRoutes.js
import express from "express";
import { listarContasReceber, listarContasPagar, listarFluxo } from "../controllers/financeiroController.js";

const router = express.Router();

router.get("/receber", listarContasReceber);
router.get("/pagar", listarContasPagar);
router.get("/fluxo", listarFluxo);

export default router;

🔹 5. server.js

Adicione a rota:

import financeiroRoutes from "./routes/financeiroRoutes.js";
app.use("/api/financeiro", financeiroRoutes);

🔹 6. Frontend — FinanceiroDashboard.jsx
import React, { useEffect, useState } from "react";
import axios from "axios";

export default function FinanceiroDashboard() {
  const [receber, setReceber] = useState([]);
  const [pagar, setPagar] = useState([]);
  const [fluxo, setFluxo] = useState([]);

  useEffect(() => {
    axios.get("http://localhost:5000/api/financeiro/receber")
      .then(res => setReceber(res.data));

    axios.get("http://localhost:5000/api/financeiro/pagar")
      .then(res => setPagar(res.data));

    axios.get("http://localhost:5000/api/financeiro/fluxo")
      .then(res => setFluxo(res.data));
  }, []);

  return (
    <section>
      <h2>💰 Financeiro</h2>

      <h3>Contas a Receber</h3>
      <table border="1" cellPadding="5" width="100%">
        <thead>
          <tr>
            <th>Cliente</th>
            <th>Descrição</th>
            <th>Valor</th>
            <th>Vencimento</th>
            <th>Status</th>
          </tr>
        </thead>
        <tbody>
          {receber.map(r => (
            <tr key={r.id}>
              <td>{r.cliente}</td>
              <td>{r.descricao}</td>
              <td>R$ {r.valor.toFixed(2)}</td>
              <td>{new Date(r.data_vencimento).toLocaleDateString()}</td>
              <td>{r.status}</td>
            </tr>
          ))}
        </tbody>
      </table>

      <h3 style={{ marginTop: 20 }}>Contas a Pagar</h3>
      <table border="1" cellPadding="5" width="100%">
        <thead>
          <tr>
            <th>Fornecedor</th>
            <th>Descrição</th>
            <th>Valor</th>
            <th>Vencimento</th>
            <th>Status</th>
          </tr>
        </thead>
        <tbody>
          {pagar.map(p => (
            <tr key={p.id}>
              <td>{p.fornecedor}</td>
              <td>{p.descricao}</td>
              <td>R$ {p.valor.toFixed(2)}</td>
              <td>{new Date(p.data_vencimento).toLocaleDateString()}</td>
              <td>{p.status}</td>
            </tr>
          ))}
        </tbody>
      </table>

      <h3 style={{ marginTop: 20 }}>Fluxo de Caixa</h3>
      <table border="1" cellPadding="5" width="100%">
        <thead>
          <tr>
            <th>Tipo</th>
            <th>Descrição</th>
            <th>Valor</th>
            <th>Data</th>
          </tr>
        </thead>
        <tbody>
          {fluxo.map(f => (
            <tr key={f.id}>
              <td>{f.tipo}</td>
              <td>{f.descricao}</td>
              <td>R$ {f.valor.toFixed(2)}</td>
              <td>{new Date(f.data_movimento).toLocaleDateString()}</td>
            </tr>
          ))}
        </tbody>
      </table>
    </section>
  );
}

🔹 7. Integração no Menu Principal

No seu menu (Sidebar ou NavBar):

<li><a href="/financeiro">Financeiro</a></li>


E no App.jsx:

import FinanceiroDashboard from "./pages/FinanceiroDashboard";
<Route path="/financeiro" element={<FinanceiroDashboard />} />


✅ Resultado Final

Visualização completa das Contas a Pagar, Contas a Receber e Fluxo de Caixa.

Totalmente integrado com Vendas e Clientes.

Código modular e separado por responsabilidade.
