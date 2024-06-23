from datetime import datetime
import sys
import msvcrt

lista_de_atividades = []
arquivo_usuarios = 'usuarios.txt'
arquivo_planejamento = 'viagens.txt'

# Mensagem de boas-vindas

def texto_verde(texto):
    return f'\033[1;32m{texto}\033[0m'

def texto_vermelho(texto):
    return f'\033[1;31m{texto}\033[0m'

def texto_amarelo(texto):
    return f'\033[1;33m{texto}\033[0m'

print('-=-' * 10)
print(f"BEM-VINDO A EMPRESA {texto_amarelo('PYVIAGENS')}")
print('-=-' * 10)

def ler_arquivo(arquivo, codificacoes=['utf-8', 'latin1', 'iso-8859-1']):
    for codificacao in codificacoes:
        try:
            with open(arquivo, 'r', encoding=codificacao) as file:
                return file.readlines()
        except UnicodeDecodeError:
            continue
    raise UnicodeDecodeError(f'Não foi possível decodificar o arquivo {arquivo} usando as codificações fornecidas.')
# Fazer com que os caracteres inseridos aparecam em "*"!
def input_senha(prompt='Senha: '):
    print(prompt, end='', flush=True)
    senha = ''
    while True:
        ch = msvcrt.getch()
        if ch in {b'\r', b'\n'}:
            print('')
            break
        elif ch == b'\x08':
            if len(senha) > 0:
                sys.stdout.write('\b \b')
                sys.stdout.flush()
                senha = senha[:-1]
        else:
            sys.stdout.write('*')
            sys.stdout.flush()
            senha += ch.decode('utf-8')
    return senha

# Codificar senha em binário
def codificar_senha(senha):
    return ' '.join(format(ord(char), '08b') for char in senha)

# Decodificar senha de binário
def decodificar_senha(senha_binaria):
    return ''.join(chr(int(bits, 2)) for bits in senha_binaria.split())

# Carregar usuários do arquivo
def carregar_usuarios():
    usuarios = {}
    try:
        linhas = ler_arquivo(arquivo_usuarios)
        for linha in linhas:
            partes = linha.strip().split(':')
            if len(partes) == 2:
                username, password_encoded = partes
                usuarios[username] = password_encoded
            else:
                print(texto_vermelho(f"Linha malformada no arquivo de usuários: {linha}"))
    except FileNotFoundError:
        print("Arquivo de usuário não encontrado. Criando um novo.")
        open(arquivo_usuarios, 'w', encoding='utf-8').close()
    return usuarios

# Cadastro de novo usuário
def novo_usuario():
    usuarios = carregar_usuarios()
    username = input('Escolha um nome para o usuário: ')
    if ':' in username:
        print(texto_vermelho("O nome de usuário não pode conter o caractere ':'."))
        return
    if username in usuarios:
        print('Nome do usuário já existe. Tente novamente!')
        return novo_usuario()
    password = input_senha('Escolha a senha: ')
    password_encoded = codificar_senha(password)
    with open(arquivo_usuarios, 'a', encoding='utf-8') as file:
        file.write(f'{username}:{password_encoded}\n')
    print(texto_verde('Usuário cadastrado com SUCESSO!'))

# Sistema de login
def login():
    usuarios = carregar_usuarios()
    username = input('Nome do usuário: ')
    password = input_senha('Senha: ')
    if username in usuarios and decodificar_senha(usuarios[username]) == password:
        print(texto_verde('Login efetuado com SUCESSO!\n'))
        print(f'Olá {texto_verde(username.upper())}, seja BEM-VINDO!')
        return 
    else:
        print(texto_vermelho('Dados inválidos! VERIFIQUE SE JÁ POSSUI CADASTRO!'))
        return None

# Carregar viagens do arquivo
def carregar_viagens():
    viagens = []
    try:
        linhas = ler_arquivo(arquivo_planejamento)
        for linha in linhas:
            viagem = linha.strip().split('|')
            if len(viagem) == 5: 
                viagens.append({
                    'usuario': viagem[0],
                    'destino': viagem[1],
                    'data_inicio': viagem[2],
                    'data_termino': viagem[3],
                    'atividades': viagem[4].split(',')
                })
            else:
                print(texto_vermelho(f"Linha malformada no arquivo de planejamento: {linha}"))
    except FileNotFoundError:
        open(arquivo_planejamento, 'w', encoding='utf-8').close()
    return viagens

# Salvar viagens no arquivo
def salvar_viagens(viagens):
    with open(arquivo_planejamento, 'w', encoding='utf-8') as file:
        for viagem in viagens:
            atividades = ','.join(viagem['atividades'])
            file.write(f"{viagem['usuario']}|{viagem['destino']}|{viagem['data_inicio']}|{viagem['data_termino']}|{atividades}\n")

# Função para verificar datas
def verificar_datas(data_inicio_str, data_termino_str):
    formato_data = "%d/%m/%Y"
    try:
        data_inicio = datetime.strptime(data_inicio_str, formato_data)
        data_termino = datetime.strptime(data_termino_str, formato_data)
        dia_atual= datetime.today()
        if data_inicio < dia_atual:
            return False, "A DATA DE INÍCIO QUE VOCÊ DIGITOU É MENOR QUE A ATUAL! TENTE NOVAMENTE!"
        if data_termino < data_inicio:
            return False, "A DATA DE TÉRMINO NÃO PODE SER MENOR DO QUE A DE INÍCIO! TENTE NOVAMENTE"
        return True, ""
    except ValueError:
        return False, "Formato de data inválido. Use DD/MM/AAAA."

# Adicionar nova viagem
def adicionar_viagem(usuario):
    destino = input('Destino da viagem: ')
    while True:
        data_inicio = input('Data de início da viagem (DD/MM/AAAA): ')
        data_termino = input('Data de término da viagem (DD/MM/AAAA): ')
        valido, mensagem = verificar_datas(data_inicio, data_termino)
        if not valido:
            print(texto_vermelho(mensagem))
        else:
            break
    atividades = input('Lista de atividades planejadas (separadas por vírgula): ')
    with open(arquivo_planejamento, 'a', encoding='utf-8') as file:
        file.write(f"{usuario}|{destino}|{data_inicio}|{data_termino}|{atividades}\n")
    print(texto_verde('VIAGEM ADICIONADA COM SUCESSO!'))

# Visualizar viagens do usuário
def visualizar_viagens(usuario):
    viagens = carregar_viagens()
    viagens_usuario = [viagem for viagem in viagens if viagem['usuario'] == usuario]
    if viagens_usuario:
        print((f"Viagens planejadas por {texto_verde(usuario.upper())}:"))
        for i, viagem in enumerate(viagens_usuario):
            print(f"\nViagem {i+1}:")
            print(f"Destino: {viagem['destino']}")
            print(f"Data de Início: {viagem['data_inicio']}")
            print(f"Data de Término: {viagem['data_termino']}")
            print("Atividades planejadas:\n")
            for atividade in viagem['atividades']:
                print(f" - {atividade}")
        return viagens_usuario
    else:
        print(texto_vermelho("NENHUMA VIAGEM ENCONTRADA!."))
        return []

# Adicionar atividade a uma viagem
def adicionar_atividade(usuario):
    viagens_usuario = visualizar_viagens(usuario)
    if not viagens_usuario:
        return
    try:
        escolha = int(input('Escolha o número da viagem para adicionar uma atividade: ')) - 1
        nova_atividade = input('Digite a nova atividade: ')
        viagens_usuario[escolha]['atividades'].append(nova_atividade)
        todas_viagens = carregar_viagens()
        for viagem in todas_viagens:
            if viagem['usuario'] == usuario and viagem['destino'] == viagens_usuario[escolha]['destino']:
                viagem['atividades'] = viagens_usuario[escolha]['atividades']
        salvar_viagens(todas_viagens)
        print(texto_verde('ATIVIDADE ADICIONADA COM SUCESSO!'))
    except (IndexError, ValueError):
        print(texto_vermelho('Seleção inválida. Tente novamente.'))

# Remover atividade de uma viagem
def remover_atividade(usuario):
    viagens_usuario = visualizar_viagens(usuario)
    if not viagens_usuario:
        return
    try:
        escolha = int(input('Escolha o número da viagem para remover uma atividade: ')) - 1
        viagem = viagens_usuario[escolha]
        for indice, atividade in enumerate(viagem['atividades'], 1):
            print(f"{indice}. {atividade}")
        escolha_atividade = int(input('Escolha o número da atividade para remover: ')) - 1
        viagem['atividades'].pop(escolha_atividade)
        todas_viagens = carregar_viagens()
        for viagem_todas in todas_viagens:
            if viagem_todas['usuario'] == usuario and viagem_todas['destino'] == viagem['destino']:
                viagem_todas['atividades'] = viagem['atividades']
        salvar_viagens(todas_viagens)
        print(texto_verde('ATIVIDADE REMOVIDA COM SUCESSO!'))
        visualizar_viagens(usuario)
    except (IndexError, ValueError):
        print(texto_vermelho('Seleção inválida. Tente novamente.'))

# Mudar destino de uma viagem
def mudar_destino(usuario):
    destino_usuario = visualizar_viagens(usuario)
    if not destino_usuario:
        return
    try:
        escolha = int(input('Escolha o número da viagem para alterar o destino: ')) - 1
        novo_destino = input('Digite o novo destino: ')
        destino_usuario[escolha]['destino'] = novo_destino
        todas_viagens = carregar_viagens()
        for viagem in todas_viagens:
            if viagem['usuario'] == usuario and viagem['data_inicio'] == destino_usuario[escolha]['data_inicio']:
                viagem['destino'] = novo_destino
        salvar_viagens(todas_viagens)
        print(texto_verde('Destino alterado com SUCESSO!'))
    except (IndexError, ValueError):
        print(texto_vermelho('Seleção inválida. Tente novamente.'))
#deletar viagens        
def excluir_viagens(usuario):
    viagens_usuario= visualizar_viagens(usuario)
    if not viagens_usuario:
        return
    try:
        escolha= int(input('Escolha o número de viagem que sedeja excluir:')) - 1
        deletar_viagem= viagens_usuario[escolha]
        todas_viagens= carregar_viagens()
        todas_viagens= [viagem for viagem in todas_viagens if not (viagem['usuario']== usuario and viagem['destino']== deletar_viagem['destino'])]
        salvar_viagens(todas_viagens)
        print(texto_verde('Viagem deletada com Sucesso!'))
    except(IndexError, ValueError):
        print(texto_vermelho('Seleção inválida. Tente Novamente'))
        
# Buscar viagens/filtragem
def filtrar_viagens(usuario):
    criterio=input('Digite o critério de busca (destino, data_inicio, data_termino):')
    valor=str(input(f'Digite o valor para {criterio}:'))
    viagens= carregar_viagens()
    resultados = [viagem for viagem in viagens if viagem['usuario']== usuario and viagem[criterio]==valor]
    if resultados:
        print(texto_verde(f'Viagens encontradas para {usuario} com {criterio} "{valor}":'))
        for viagem in resultados:
            print(f"\nDestino: {viagem['destino']}")
            print(f"\Data_Início: {viagem['data_inicio']}")
            print(f"\Data_Término: {viagem['data_termino']}")
            print("Atividades planejadas:")
            for atividade in viagem['atividades']:
                print(f' - {atividade}')
            visualizar_viagens(usuario)
    else:
        print(texto_vermelho('Nenhuma viagem encontrada com esses critério.'))
        
def lobby(usuario):
    print("\n1. Adicionar Atividade")
    print("2. Remover Atividade")
    print("4. Deletar Viagens")
    print("5. Filtrar Viagens")
    
    escolha = input("Escolha uma opção: ")
    if escolha == "1":
        adicionar_atividade(usuario)
    elif escolha == "2":
        remover_atividade(usuario)
    elif escolha == "3":
        mudar_destino(usuario)
    elif escolha == "4":
        excluir_viagens(usuario)
    elif escolha == "5":
        filtrar_viagens(usuario)
        
# Loop principal
def Menu_Principal():
    while True:
        print("\n1. Novo Usuário")
        print("2. Login")
        print("3. Sair")
        escolha = input("Escolha uma opção: ")
        if escolha == '1':
            novo_usuario()
        elif escolha == '2':
            usuario = login()
            if usuario:
                while True:
                    print("\n1. Adicionar Viagem")
                    print("2. Visualizar Viagens")
                    print("3. Logout")
                    escolha_usuario = input("Escolha uma opção: ")
                    if escolha_usuario == '1':
                        adicionar_viagem(usuario)
                    elif escolha_usuario == '2':
                        visualizar_viagens(usuario)
                        lobby(usuario)
                    elif escolha_usuario == '3':
                        print(texto_verde('Usuário Desconectado!'))
                        break
                    else:
                        print(texto_vermelho("Opção inválida. Tente novamente."))
        elif escolha == '3':
            break
        else:
            print(texto_vermelho("Opção inválida. Tente novamente."))

if __name__== "__main__":
    Menu_Principal()
