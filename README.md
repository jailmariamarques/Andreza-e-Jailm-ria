import csv
import datetime
import ipywidgets as widgets
from IPython.display import display, clear_output
import smtplib
from email.mime.multipart import MIMEMultipart
from email.mime.text import MIMEText

class Catalogo:
    def __init__(self):
        self.arquivo_catalogo = "catalogo.csv"
        self.arquivo_historico = "historico.csv"
        self.capacidade_total = 1000  # Capacidade total do estoque (ajuste conforme necessário)
        self.create_widgets()
        self.check_stock_levels()

    def create_widgets(self):
        self.out = widgets.Output()

        # Widgets for adding items
        self.add_codigo = widgets.Text(description="Código:")
        self.add_descricao = widgets.Text(description="Descrição:")
        self.add_quantidade = widgets.IntText(description="Quantidade:")
        self.add_button = widgets.Button(description="Adicionar Item")
        self.add_button.on_click(self.adicionar_item)

        # Widgets for searching items
        self.search_codigo = widgets.Text(description="Código:")
        self.search_button = widgets.Button(description="Pesquisar Item")
        self.search_button.on_click(self.pesquisar_item)

        # Widgets for updating quantity
        self.update_codigo = widgets.Text(description="Código:")
        self.update_quantidade = widgets.IntText(description="Nova Quantidade:")
        self.update_button = widgets.Button(description="Atualizar Quantidade")
        self.update_button.on_click(self.atualizar_quantidade)

        # Widget for displaying catalog
        self.display_catalog_button = widgets.Button(description="Exibir Catálogo")
        self.display_catalog_button.on_click(self.exibir_catalogo)

        # Widget for displaying history
        self.display_history_button = widgets.Button(description="Exibir Histórico")
        self.display_history_button.on_click(self.exibir_historico)

        # Widgets for removing items
        self.remove_codigo = widgets.Text(description="Código:")
        self.remove_button = widgets.Button(description="Remover Item")
        self.remove_button.on_click(self.remover_item)

    def adicionar_item(self, b):
        codigo = self.add_codigo.value
        descricao = self.add_descricao.value
        quantidade = self.add_quantidade.value

        with open(self.arquivo_catalogo, "a", newline="") as arquivo:
            writer = csv.writer(arquivo)
            writer.writerow([codigo, descricao, quantidade])

        with self.out:
            clear_output()
            print("Item adicionado com sucesso!")

        self.check_stock_levels()

    def pesquisar_item(self, b):
        codigo = self.search_codigo.value

        try:
            with open(self.arquivo_catalogo, "r") as arquivo:
                reader = csv.reader(arquivo)
                for linha in reader:
                    if linha[0] == codigo:
                        with self.out:
                            clear_output()
                            print("Descrição:", linha[1])
                            print("Quantidade disponível:", linha[2])
                        return
            with self.out:
                clear_output()
                print("Item não encontrado.")
        except FileNotFoundError:
            with self.out:
                clear_output()
                print("Arquivo do catálogo não encontrado.")
        except Exception as e:
            with self.out:
                clear_output()
                print("Ocorreu um erro:", e)

    def atualizar_quantidade(self, b):
        codigo = self.update_codigo.value
        nova_quantidade = self.update_quantidade.value

        try:
            linhas_atualizadas = []
            with open(self.arquivo_catalogo, "r") as arquivo:
                reader = csv.reader(arquivo)
                for linha in reader:
                    if linha[0] == codigo:
                        linha[2] = nova_quantidade
                    linhas_atualizadas.append(linha)

            with open(self.arquivo_catalogo, "w", newline="") as arquivo:
                writer = csv.writer(arquivo)
                writer.writerows(linhas_atualizadas)

            with self.out:
                clear_output()
                print("Quantidade atualizada com sucesso!")
        except FileNotFoundError:
            with self.out:
                clear_output()
                print("Arquivo do catálogo não encontrado.")
        except Exception as e:
            with self.out:
                clear_output()
                print("Ocorreu um erro:", e)

        self.check_stock_levels()

    def exibir_catalogo(self, b):
        try:
            with open(self.arquivo_catalogo, "r") as arquivo:
                reader = csv.reader(arquivo)
                with self.out:
                    clear_output()
                    for linha in reader:
                        print("Código:", linha[0])
                        print("Descrição:", linha[1])
                        print("Quantidade disponível:", linha[2])
                        print()
        except FileNotFoundError:
            with self.out:
                clear_output()
                print("Arquivo do catálogo não encontrado.")
        except Exception as e:
            with self.out:
                clear_output()
                print("Ocorreu um erro:", e)

    def exibir_historico(self, b):
        try:
            with open(self.arquivo_historico, "r") as arquivo:
                reader = csv.reader(arquivo)
                with self.out:
                    clear_output()
                    for linha in reader:
                        data = datetime.datetime.strptime(linha[0], "%Y-%m-%d %H:%M:%S")
                        descricao = linha[1]
                        quantidade = linha[2]
                        print("Data e hora:", data)
                        print("Descrição:", descricao)
                        print("Quantidade:", quantidade)
                        print()
        except FileNotFoundError:
            with self.out:
                clear_output()
                print("Arquivo do histórico não encontrado.")
        except Exception as e:
            with self.out:
                clear_output()
                print("Ocorreu um erro:", e)

    def remover_item(self, b):
        codigo = self.remove_codigo.value
        item_encontrado = False

        try:
            linhas_atualizadas = []
            with open(self.arquivo_catalogo, "r") as arquivo:
                reader = csv.reader(arquivo)
                for linha in reader:
                    if linha[0] == codigo:
                        item_encontrado = True
                    else:
                        linhas_atualizadas.append(linha)

            if item_encontrado:
                with open(self.arquivo_catalogo, "w", newline="") as arquivo:
                    writer = csv.writer(arquivo)
                    writer.writerows(linhas_atualizadas)
                with self.out:
                    clear_output()
                    print("Item removido com sucesso!")
            else:
                with self.out:
                    clear_output()
                    print("Item não encontrado.")
        except FileNotFoundError:
            with self.out:
                clear_output()
                print("Arquivo do catálogo não encontrado.")
        except Exception as e:
            with self.out:
                clear_output()
                print("Ocorreu um erro:", e)

        self.check_stock_levels()

    def check_stock_levels(self):
        try:
            total_quantidade = 0
            with open(self.arquivo_catalogo, "r") as arquivo:
                reader = csv.reader(arquivo)
                for linha in reader:
                    total_quantidade += int(linha[2])
            
            if total_quantidade < 0.3 * self.capacidade_total:
                self.enviar_aviso_email(total_quantidade)
        except FileNotFoundError:
            with self.out:
                clear_output()
                print("Arquivo do catálogo não encontrado.")
        except Exception as e:
            with self.out:
                clear_output()
                print("Ocorreu um erro ao verificar o nível de estoque:", e)

    def enviar_aviso_email(self, quantidade_atual):
        EMAIL_ACCOUNT = 'seu_email@exemplo.com'
        PASSWORD = 'sua_senha'
        FORWARD_TO_EMAIL = 'destinatario@exemplo.com'
        AVISO = "Aviso: Almoxarifado com capacidade menor que 30%"

        msg = MIMEMultipart()
        msg['From'] = EMAIL_ACCOUNT
        msg['To'] = FORWARD_TO_EMAIL
        msg['Subject'] = "Aviso de Baixo Estoque"
        body = f"O estoque atual é de {quantidade_atual} itens, o que é menor que 30% da capacidade total."
        msg.attach(MIMEText(body, 'plain'))

        try:
            with smtplib.SMTP('smtp.exemplo.com', 587) as server:
                server.starttls()
                server.login(EMAIL_ACCOUNT, PASSWORD)
                server.sendmail(EMAIL_ACCOUNT, FORWARD_TO_EMAIL, msg.as_string())
            with self.out:
                clear_output()
                print("E-mail de aviso enviado com sucesso!")
        except Exception as e:
            with self.out:
                clear_output()
                print("Ocorreu um erro ao enviar o e-mail:", e)

    def display_menu(self):
        menu_box = widgets.VBox([
            widgets.HBox([self.add_codigo, self.add_descricao, self.add_quantidade, self.add_button]),
            widgets.HBox([self.search_codigo, self.search_button]),
            widgets.HBox([self.update_codigo, self.update_quantidade, self.update_button]),
            self.display_catalog_button,
            self.display_history_button,
            widgets.HBox([self.remove_codigo, self.remove_button]),
            self.out
        ])
        display(menu_box)

# Cria uma instância da classe Catalogo
catalogo = Catalogo()

# Exibe o menu
catalogo.display_menu()
