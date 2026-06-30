# bancofake
import os
import json
from datetime import datetime
import customtkinter as ctk
from tkinter import messagebox

# Configuração global do visual moderno
ctk.set_appearance_mode("dark")  # Modo escuro nativo do Nu
ctk.set_default_color_theme("purple")  # Tema roxo do Nubank

DATA_FILE = "usuarios_banco.json"

def carregar_usuarios():
    if not os.path.exists(DATA_FILE):
        return {}
    try:
        with open(DATA_FILE, "r", encoding="utf-8") as f:
            return json.load(f)
    except Exception:
        return {}

def salvar_usuarios(usuarios):
    with open(DATA_FILE, "w", encoding="utf-8") as f:
        json.dump(usuarios, f, indent=4, ensure_ascii=False)

class NubankSimulador:
    def __init__(self, root):
        self.root = root
        self.root.title("Nubank")
        self.root.geometry("420x650")
        self.root.resizable(False, False)
        
        self.usuarios = carregar_usuarios()
        self.usuario_logado = None
        self.tela_login()

    def limpar_tela(self):
        for widget in self.root.winfo_children():
            widget.destroy()

    def registrar_transacao(self, tipo, valor):
        data_hora = datetime.now().strftime("%d/%m/%Y %H:%M")
        extrato_item = f"[{data_hora}] {tipo}: R$ {valor:.2f}"
        self.usuarios[self.usuario_logado]["extrato"].append(extrato_item)
        salvar_usuarios(self.usuarios)

    def tela_login(self):
        self.limpar_tela()
        
        # Logo Nu
        lbl_logo = ctk.CTkLabel(self.root, text="nu", font=("Helvetica", 54, "bold"), text_color="#820AD1")
        lbl_logo.pack(pady=(50, 10))
        
        lbl_boas_vidas = ctk.CTkLabel(self.root, text="Olá! Que bom ter você de volta.", font=("Helvetica", 16))
        lbl_boas_vidas.pack(pady=(0, 40))
        
        # Inputs
        self.ent_cpf = ctk.CTkEntry(self.root, placeholder_text="Digite seu CPF (apenas números)", width=320, height=45)
        self.ent_cpf.pack(pady=10)
        
        self.ent_senha = ctk.CTkEntry(self.root, placeholder_text="Senha", show="*", width=320, height=45)
        self.ent_senha.pack(pady=10)
        
        # Botões
        btn_entrar = ctk.CTkButton(self.root, text="Entrar", font=("Helvetica", 14, "bold"), fg_color="#820AD1", hover_color="#6708a6", width=320, height=45, command=self.acao_login)
        btn_entrar.pack(pady=(30, 10))
        
        btn_ir_cadastro = ctk.CTkButton(self.root, text="Criar uma conta nova", font=("Helvetica", 12, "underline"), fg_color="transparent", text_color="white", hover=False, command=self.tela_cadastro)
        btn_ir_cadastro.pack(pady=10)

    def tela_cadastro(self):
        self.limpar_tela()
        
        lbl_titulo = ctk.CTkLabel(self.root, text="Crie sua conta Nu", font=("Helvetica", 24, "bold"), text_color="#820AD1")
        lbl_titulo.pack(pady=(40, 30))
        
        self.ent_cad_nome = ctk.CTkEntry(self.root, placeholder_text="Nome Completo", width=320, height=45)
        self.ent_cad_nome.pack(pady=10)
        
        self.ent_cad_cpf = ctk.CTkEntry(self.root, placeholder_text="CPF (apenas números)", width=320, height=45)
        self.ent_cad_cpf.pack(pady=10)
        
        self.ent_cad_senha = ctk.CTkEntry(self.root, placeholder_text="Crie uma Senha", show="*", width=320, height=45)
        self.ent_cad_senha.pack(pady=10)
        
        btn_cadastrar = ctk.CTkButton(self.root, text="Cadastrar", font=("Helvetica", 14, "bold"), fg_color="#820AD1", hover_color="#6708a6", width=320, height=45, command=self.acao_cadastro)
        btn_cadastrar.pack(pady=(30, 10))
        
        btn_voltar = ctk.CTkButton(self.root, text="Voltar para o Login", font=("Helvetica", 12, "underline"), fg_color="transparent", text_color="white", hover=False, command=self.tela_login)
        btn_voltar.pack(pady=10)

    def tela_principal(self):
        self.limpar_tela()
        dados = self.usuarios[self.usuario_logado]
        
        # Cabeçalho
        lbl_ola = ctk.CTkLabel(self.root, text=f"Olá, {dados['nome']}", font=("Helvetica", 20, "bold"))
        lbl_ola.pack(anchor="w", padx=30, pady=(30, 10))
        
        # Card de Saldo Branco (Estilo Nu)
        card_conta = ctk.CTkFrame(self.root, fg_color="#1E1E1E", border_width=1, border_color="#333333", corner_radius=12)
        card_conta.pack(fill="x", padx=25, pady=10, ipady=10)
        
        lbl_tit_saldo = ctk.CTkLabel(card_conta, text="Conta / Saldo disponível", font=("Helvetica", 13), text_color="#A9A9A9")
        lbl_tit_saldo.pack(anchor="w", padx=20, pady=(10, 2))
        
        self.lbl_saldo = ctk.CTkLabel(card_conta, text=f"R$ {dados['saldo']:.2f}", font=("Helvetica", 26, "bold"), text_color="white")
        self.lbl_saldo.pack(anchor="w", padx=20, pady=(0, 10))
        
        # Inputs de Operações
        frame_operacoes = ctk.CTkFrame(self.root, fg_color="transparent")
        frame_operacoes.pack(fill="x", padx=25, pady=15)
        
        self.ent_valor_mov = ctk.CTkEntry(frame_operacoes, placeholder_text="R$ 0,00", font=("Helvetica", 16), height=45)
        self.ent_valor_mov.pack(fill="x", pady=(0, 10))
        
        # Botões lado a lado (Depositar e Pix)
        frame_botoes = ctk.CTkFrame(frame_operacoes, fg_color="transparent")
        frame_botoes.pack(fill="x")
        
        btn_depositar = ctk.CTkButton(frame_botoes, text="Depositar", font=("Helvetica", 12, "bold"), fg_color="#242424", text_color="white", hover_color="#333333", height=40, width=175, command=self.acao_deposito)
        btn_depositar.pack(side="left", padx=(0, 5))
        
        btn_pix = ctk.CTkButton(frame_botoes, text="Enviar Pix", font=("Helvetica", 12, "bold"), fg_color="#820AD1", hover_color="#6708a6", height=40, width=175, command=self.acao_transferencia)
        btn_pix.pack(side="right", padx=(5, 0))
        
        # Seção de Extrato / Histórico
        lbl_tit_extrato = ctk.CTkLabel(self.root, text="Histórico de transações", font=("Helvetica", 14, "bold"), text_color="#A9A9A9")
        lbl_tit_extrato.pack(anchor="w", padx=30, pady=(15, 5))
        
        self.txt_extrato = ctk.CTkTextbox(self.root, height=150, fg_color="#1E1E1E", border_width=1, border_color="#333333", font=("Courier", 12))
        self.txt_extrato.pack(fill="x", padx=25, pady=5)
        self.atualizar_extrato_tela()
        
        # Botão Sair
        btn_sair = ctk.CTkButton(self.root, text="Sair da Conta", font=("Helvetica", 12, "bold"), fg_color="#D93B3B", hover_color="#b32d2d", height=40, command=self.tela_login)
        btn_sair.pack(side="bottom", fill="x", padx=25, pady=25)

    def atualizar_extrato_tela(self):
        self.txt_extrato.configure(state="normal")
        self.txt_extrato.delete("1.0", ctk.END)
        extrato = self.usuarios[self.usuario_logado].get("extrato", [])
        
        if not extrato:
            self.txt_extrato.insert(ctk.END, "Nenhuma movimentação realizada.")
        else:
            for item in reversed(extrato):  # Mostra as mais recentes primeiro
                self.txt_extrato.insert(ctk.END, f"{item}\n")
                
        self.txt_extrato.configure(state="disabled")

    # --- LÓGICA DE NEGÓCIO REVISADA ---
    def acao_login(self):
        cpf = self.ent_cpf.get().strip()
        senha = self.ent_senha.get().strip()
        if cpf in self.usuarios and self.usuarios[cpf]["senha"] == senha:
            self.usuario_logado = cpf
            self.tela_principal()
        else:
            messagebox.showerror("Erro de Login", "CPF ou senha incorretos.")

    def acao_cadastro(self):
        nome = self.ent_cad_nome.get().strip()
        cpf = self.ent_cad_cpf.get().strip()
        senha = self.ent_cad_senha.get().strip()
        
        if not nome or not cpf or not senha:
            messagebox.showwarning("Aviso", "Todos os campos devem ser preenchidos.")
            return
        if cpf in self.usuarios:
            messagebox.showerror("Erro", "Este CPF já possui cadastro.")
            return
            
        self.usuarios[cpf] = {
            "nome": nome, 
            "senha": senha, 
            "saldo": 500.00,
            "extrato": ["[Cadastro] Bônus de Boas-Vindas: R$ 500.00"]
        }
        salvar_usuarios(self.usuarios)
        messagebox.showinfo("Sucesso", "Conta criada com sucesso!")
        self.tela_login()

    def acao_deposito(self):
        try:
            valor = float(self.ent_valor_mov.get())
            if valor <= 0: raise ValueError
        except ValueError:
            messagebox.showwarning("Valor Inválido", "Digite um valor maior que zero.")
            return
            
        self.usuarios[self.usuario_logado]["saldo"] += valor
        self.registrar_transacao("Depósito", valor)
        
        self.lbl_saldo.configure(text=f"R$ {self.usuarios[self.usuario_logado]['saldo']:.2f}")
        self.ent_valor_mov.delete(0, ctk.END)
        self.atualizar_extrato_tela()
        messagebox.showinfo("Sucesso", f"Depósito de R$ {valor:.2f} concluído!")

    def acao_transferencia(self):
        try:
            valor = float(self.ent_valor_mov.get())
            if valor <= 0: raise ValueError
        except ValueError:
            messagebox.showwarning("Valor Inválido", "Digite um valor maior que zero.")
            return
            
        saldo_atual = self.usuarios[self.usuario_logado]["saldo"]
        if valor > saldo_atual:
            messagebox.showerror("Saldo Insuficiente", "Você não tem saldo suficiente.")
            return
            
        self.usuarios[self.usuario_logado]["saldo"] -= valor
        self.registrar_transacao("Envio Pix", valor)
        
        self.lbl_saldo.configure(text=f"R$ {self.usuarios[self.usuario_logado]['saldo']:.2f}")
        self.ent_valor_mov.delete(0, ctk.END)
        self.atualizar_extrato_tela()
        messagebox.showinfo("Pix Enviado", f"Pix de R$ {valor:.2f} enviado!")

if __name__ == "__main__":
    app_root = ctk.CTk()
    simulador = NubankSimulador(app_root)
    app_root.mainloop()
