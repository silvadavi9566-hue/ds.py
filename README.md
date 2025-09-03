# ds.py
aa
import tkinter as tk
from tkinter import ttk, messagebox
from datetime import datetime, timedelta

class RevezamentoTurmaD:
    def __init__(self, root):
        self.root = root
        self.root.title("REVEZAMENTO TURMA D")
        self.root.geometry("1200x700")
        
        # Dados iniciais
        self.dados_operadores = [
            {"nome": "Operador 1", "caminhao": "CAM-001", "inicio_janta": "18:00", "fim_janta": "18:30"},
            {"nome": "Operador 2", "caminhao": "CAM-002", "inicio_janta": "18:15", "fim_janta": "18:45"},
            {"nome": "Operador 3", "caminhao": "CAM-003", "inicio_janta": "19:00", "fim_janta": "19:30"},
        ]
        
        self.setup_ui()
        self.update_timers()
        
    def setup_ui(self):
        # Configurar estilo
        self.style = ttk.Style()
        self.style.configure("Treeview", font=('Arial', 14), rowheight=40)
        self.style.configure("Treeview.Heading", font=('Arial', 14, 'bold'))
        self.style.configure("Atrasado.Treeview", foreground="red", font=('Arial', 14, 'bold'))
        
        # Cabeçalho
        header_frame = tk.Frame(self.root, bg="#333", padx=10, pady=15)
        header_frame.pack(fill=tk.X)
        
        tk.Label(header_frame, text="REVEZAMENTO TURMA D", font=("Arial", 20, "bold"), 
                fg="white", bg="#333").pack(side=tk.LEFT)
        
        self.relogio = tk.Label(header_frame, font=("Arial", 14), fg="white", bg="#333")
        self.relogio.pack(side=tk.RIGHT)
        self.update_clock()
        
        # Controles
        control_frame = tk.Frame(self.root, padx=10, pady=10)
        control_frame.pack(fill=tk.X)
        
        tk.Button(control_frame, text="+ Adicionar Operador", command=self.adicionar_operador, 
                bg="#4CAF50", fg="white", font=('Arial', 12)).pack(side=tk.LEFT, padx=5)
        tk.Button(control_frame, text="− Remover Selecionado", command=self.remover_operador,
                bg="#f44336", fg="white", font=('Arial', 12)).pack(side=tk.LEFT, padx=5)
        tk.Button(control_frame, text="💾 Salvar Alterações", command=self.salvar_dados,
                bg="#2196F3", fg="white", font=('Arial', 12)).pack(side=tk.RIGHT, padx=5)
        
        # Tabela editável
        columns = ("nome", "caminhao", "inicio_janta", "fim_janta", "tempo_restante")
        self.tree = ttk.Treeview(self.root, columns=columns, show="headings", height=15)
        
        # Configurar colunas
        col_configs = [
            ("Operador", 250, tk.W),
            ("Caminhão", 200, tk.CENTER),
            ("Início Janta", 200, tk.CENTER),
            ("Fim Janta", 200, tk.CENTER),
            ("Status", 300, tk.CENTER)
        ]
        
        for idx, (text, width, anchor) in enumerate(col_configs):
            self.tree.heading(columns[idx], text=text)
            self.tree.column(columns[idx], width=width, anchor=anchor)
        
        self.tree.pack(fill=tk.BOTH, expand=True, padx=15, pady=10)
        
        # Adicionar dados iniciais
        self.carregar_dados()
        
        # Configurar edição por duplo clique
        self.tree.bind("<Double-1>", self.editar_celula)
        
    def carregar_dados(self):
        self.tree.delete(*self.tree.get_children())
        for i, operador in enumerate(self.dados_operadores):
            # Alternar cores entre cinza e branco
            bg_color = '#f0f0f0' if i % 2 == 0 else 'white'
            self.tree.insert("", tk.END, values=(
                operador["nome"],
                operador["caminhao"],
                operador["inicio_janta"],
                operador["fim_janta"],
                "Calculando..."
            ), tags=(bg_color,))
        
        # Configurar tags para cores alternadas
        self.tree.tag_configure('#f0f0f0', background='#f0f0f0', font=('Arial', 14))
        self.tree.tag_configure('white', background='white', font=('Arial', 14))
    
    def editar_celula(self, event):
        item = self.tree.selection()[0]
        column = self.tree.identify_column(event.x)
        col_index = int(column[1:]) - 1
        
        if col_index == 4:  # Não editar coluna de status
            return
        
        valores = list(self.tree.item(item, 'values'))
        valor_atual = valores[col_index]
        
        edit_window = tk.Toplevel(self.root)
        edit_window.title(f"Editar {self.tree.heading(column)['text']}")
        edit_window.geometry("400x150")
        
        tk.Label(edit_window, text=f"Novo valor para {self.tree.heading(column)['text']}:", 
                font=('Arial', 12)).pack(pady=10)
        
        entry = tk.Entry(edit_window, width=25, font=('Arial', 14))
        entry.pack(pady=10)
        entry.insert(0, valor_atual)
        entry.focus_set()
        
        def confirmar_edicao():
            novo_valor = entry.get()
            valores[col_index] = novo_valor
            self.tree.item(item, values=valores)
            edit_window.destroy()
            self.atualizar_dados_memoria()
        
        btn_confirmar = tk.Button(edit_window, text="✔ Confirmar", command=confirmar_edicao, 
                                bg="#4CAF50", fg="white", font=('Arial', 12))
        btn_confirmar.pack(pady=5)
        edit_window.bind("<Return>", lambda e: confirmar_edicao())
    
    def atualizar_dados_memoria(self):
        self.dados_operadores = []
        for item in self.tree.get_children():
            valores = self.tree.item(item, 'values')
            self.dados_operadores.append({
                "nome": valores[0],
                "caminhao": valores[1],
                "inicio_janta": valores[2],
                "fim_janta": valores[3]
            })
    
    def adicionar_operador(self):
        novo_operador = {
            "nome": "Novo Operador",
            "caminhao": "CAM-000",
            "inicio_janta": datetime.now().strftime("%H:%M"),
            "fim_janta": (datetime.now() + timedelta(minutes=30)).strftime("%H:%M")
        }
        self.dados_operadores.append(novo_operador)
        self.carregar_dados()
    
    def remover_operador(self):
        selecionados = self.tree.selection()
        if not selecionados:
            messagebox.showwarning("Aviso", "Nenhum operador selecionado!")
            return
        
        item = selecionados[0]
        nome_operador = self.tree.item(item, 'values')[0]
        
        if messagebox.askyesno("Confirmar", f"Remover o operador {nome_operador}?"):
            index = self.tree.index(item)
            del self.dados_operadores[index]
            self.carregar_dados()
    
    def salvar_dados(self):
        self.atualizar_dados_memoria()
        messagebox.showinfo("Sucesso", "Todos os dados foram salvos!")
    
    def update_clock(self):
        agora = datetime.now().strftime("%H:%M:%S - %d/%m/%Y")
        self.relogio.config(text=agora)
        self.root.after(1000, self.update_clock)
    
    def update_timers(self):
        agora = datetime.now()
        
        for item in self.tree.get_children():
            valores = list(self.tree.item(item, 'values'))
            inicio_str = valores[2]
            fim_str = valores[3]
            
            try:
                inicio_janta = datetime.strptime(inicio_str, "%H:%M")
                inicio_janta = inicio_janta.replace(year=agora.year, month=agora.month, day=agora.day)
                
                fim_janta = datetime.strptime(fim_str, "%H:%M")
                fim_janta = fim_janta.replace(year=agora.year, month=agora.month, day=agora.day)
                
                if agora < inicio_janta:
                    tempo_espera = inicio_janta - agora
                    tempo_str = str(tempo_espera).split(".")[0]
                    status = f"🕒 Aguardando início ({tempo_str})"
                    self.tree.item(item, tags=(self.tree.item(item, 'tags')[0],))
                elif inicio_janta <= agora < fim_janta:
                    tempo_restante = fim_janta - agora
                    tempo_str = str(tempo_restante).split(".")[0]
                    
                    if tempo_restante < timedelta(minutes=10):
                        status = f"⚠ Finalizando em {tempo_str}"
                    else:
                        status = f"🍴 Em janta ({tempo_str} restantes)"
                    self.tree.item(item, tags=(self.tree.item(item, 'tags')[0],))
                else:  # Passou do horário de fim
                    tempo_atraso = agora - fim_janta
                    tempo_str = str(tempo_atraso).split(".")[0]
                    status = f"❗ EM ATRASO (+{tempo_str})"
                    self.tree.item(item, tags=(self.tree.item(item, 'tags')[0], 'atrasado'))
                
                valores[4] = status
                self.tree.item(item, values=valores)
                
            except ValueError:
                pass
        
        # Configurar tag para atraso
        self.tree.tag_configure('atrasado', foreground='red', font=('Arial', 14, 'bold'))
        self.root.after(1000, self.update_timers)

if __name__ == "__main__":
    root = tk.Tk()
    app = RevezamentoTurmaD(root)
    root.mainloop()
