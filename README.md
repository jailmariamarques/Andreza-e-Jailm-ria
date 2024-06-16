import csv
import datetime
import imaplib
import smtplib
from email.mime.multipart import MIMEMultipart
from email.mime.text import MIMEText
from email.parser import BytesParser
from email.policy import default


class Catalogo:
    def __init__(self):
        self.arquivo_catalogo = "catalogo.csv"
        self.arquivo_historico = "historico.csv"

    def adicionar_item(self):
        codigo = input("Digite o código do item: ")
        descricao = input("Digite a descrição do item: ")
        quantidade = input("Digite a quantidade disponível: ")

        with open(self.arquivo_catalogo, "a", newline="") as arquivo:
            writer = csv.writer(arquivo)
            writer.writerow([codigo, descricao, quantidade])

        print("Item adicionado com sucesso!")

    def pesquisar_item(self):
        codigo = input("Digite o código do item: ")

        try:
            with open(self.arquivo_catalogo, "r") as arquivo:
                reader = csv.reader(arquivo)
                for linha in reader:
                    if linha[0] == codigo:
                        print("Descrição:", linha[1])
                        print("Quantidade disponível:", linha[2])
                        return
            print("Item não encontrado.")
        except FileNotFoundError:
            print("Arquivo do catálogo não encontrado.")
        except Exception as e:
            print("Ocorreu um erro:", e)

    def atualizar_quantidade(self):
        codigo = input("Digite o código do item: ")
        nova_quantidade = input("Digite a nova quantidade disponível: ")

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

            print("Quantidade atualizada com sucesso!")
        except FileNotFoundError:
            print("Arquivo do catálogo não encontrado.")
        except Exception as e:
            print("Ocorreu um erro:", e)

    def exibir_catalogo(self):
        try:
            with open(self.arquivo_catalogo, "r") as arquivo:
                reader = csv.reader(arquivo)
                for linha in reader:
                    print("Código:", linha[0])
                    print("Descrição:", linha[1])
                    print("Quantidade disponível:", linha[2])
                    print()
        except FileNotFoundError:
            print("Arquivo do catálogo não encontrado.")
        except Exception as e:
            print("Ocorreu um erro:", e)

    def exibir_historico(self):
        try:
            with open(self.arquivo_historico, "r") as arquivo:
                reader = csv.reader(arquivo)
                for linha in reader:
                    data = datetime.datetime.strptime(linha[0], "%Y-%m-%d %H:%M:%S")
                    descricao = linha[1]
                    quantidade = linha[2]
                    print("Data e hora:", data)
                    print("Descrição:", descricao)
                    print("Quantidade:", quantidade)
                    print()
        except FileNotFoundError:
            print("Arquivo do histórico não encontrado.")
        except Exception as e:
            print("Ocorreu um erro:", e)

    def remover_item(self):
        codigo = input("Digite o código do item a ser removido: ")
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
                print("Item removido com sucesso!")
            else:
                print("Item não encontrado.")
        except FileNotFoundError:
            print("Arquivo do catálogo não encontrado.")
        except Exception as e:
            print("Ocorreu um erro:", e)

    def menu(self):
        while True:
            print("1. Adicionar item")
            print("2. Pesquisar item")
            print("3. Atualizar quantidade")
            print("4. Exibir catálogo")
            print("5. Exibir histórico")
            print("6. Remover item")
            print("7. Sair")

            opcao = input("Digite a opção desejada: ")

            if opcao == "1":
                self.adicionar_item()
            elif opcao == "2":
                self.pesquisar_item()
            elif opcao == "3":
                self.atualizar_quantidade()
            elif opcao == "4":
                self.exibir_catalogo()
            elif opcao == "5":
                self.exibir_historico()
            elif opcao == "6":
                self.remover_item()
            elif opcao == "7":
                break
            else:
                print("Opção inválida. Tente novamente.")

            print()


# Configurações do e-mail
IMAP_SERVER = 'imap.exemplo.com'
IMAP_PORT = 993
SMTP_SERVER = 'smtp.exemplo.com'
SMTP_PORT = 587
EMAIL_ACCOUNT = 'seu_email@exemplo.com'
PASSWORD = 'sua_senha'
FORWARD_TO_EMAIL = 'destinatario@exemplo.com'
AVISO = "Aviso: Almoxarifado com capacidade menor que 30%"


def check_email():
    mail = imaplib.IMAP4_SSL(IMAP_SERVER, IMAP_PORT)
    mail.login(EMAIL_ACCOUNT, PASSWORD)
    mail.select('inbox')
    status, messages = mail.search(None, '(UNSEEN)')
    email_ids = messages[0].split()

    for email_id in email_ids:
        status, msg_data = mail.fetch(email_id, '(RFC822)')
        raw_email = msg_data[0][1]
        msg = BytesParser(policy=default).parsebytes(raw_email)

        forward_email(msg)

    mail.logout()


def forward_email(original_msg):
    msg = MIMEMultipart()
    msg['From'] = EMAIL_ACCOUNT
    msg['To'] = FORWARD_TO_EMAIL
    msg['Subject'] = "Fwd: " + original_msg['Subject']
    body = AVISO + "\n\n" + original_msg.get_body(preferencelist=('plain',)).get_content()

    msg.attach(MIMEText(body, 'plain'))
    with smtplib.SMTP(SMTP_SERVER, SMTP_PORT) as server:
        server.starttls()
        server.login(EMAIL_ACCOUNT, PASSWORD)
        server.sendmail(EMAIL_ACCOUNT, FORWARD_TO_EMAIL, msg.as_string())


if __name__ == '__main__':
    check_email()

    # Cria uma instância da classe Catalogo
    catalogo = Catalogo()

    # Executa o programa
    catalogo.menu()
