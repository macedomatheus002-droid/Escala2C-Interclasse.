# Escala2C-Interclasse.
# -*- coding: utf-8 -*-
"""Escala2C - Sistema de acompanhamento do campeonato interclasse.

Estruturas usadas:
- classificacao: [PJ, V, E, D, GP, GC, SG, PTS]
- jogador: [codigo, posicao, presente]
"""

MAX_TIMES = 16

times = []
classificacao = []
confrontos = []
jogadores_por_time = []
campeonato_iniciado = False


def ler_inteiro(mensagem):
    while True:
        try:
            return int(input(mensagem))
        except ValueError:
            print("ERRO: digite apenas um número inteiro.")


def ler_opcao(mensagem, minimo, maximo):
    while True:
        valor = ler_inteiro(mensagem)
        if minimo <= valor <= maximo:
            return valor
        print(f"ERRO: digite um valor entre {minimo} e {maximo}.")


def confirmar(mensagem):
    while True:
        resposta = input(f"{mensagem} (S/N): ").strip().upper()
        if resposta in ("S", "N"):
            return resposta
        print("ERRO: digite somente S ou N.")


def ler_gols(mensagem):
    while True:
        gols = ler_inteiro(mensagem)
        if gols >= 0:
            return gols
        print("ERRO: a quantidade de gols não pode ser negativa.")


def texto_tem_caracteres_validos(texto):
    return all(c.isalnum() or c in " -" for c in texto)


def normalizar_espacos(texto):
    return " ".join(texto.split())


def ler_nome_time(indice_edicao=None):
    while True:
        nome = normalizar_espacos(input("Nome do time: ").strip())
        if not nome:
            print("ERRO: o nome do time não pode ficar vazio.")
            continue
        if len(nome) < 2:
            print("ERRO: o nome deve possuir pelo menos 2 caracteres.")
            continue
        if len(nome) > 30:
            print("ERRO: o nome deve possuir no máximo 30 caracteres.")
            continue
        if not texto_tem_caracteres_validos(nome):
            print("ERRO: use apenas letras, números, espaços ou hífen.")
            continue

        duplicado = any(
            i != indice_edicao and time.casefold() == nome.casefold()
            for i, time in enumerate(times)
        )
        if duplicado:
            print("ERRO: já existe um time com esse nome.")
            continue
        return nome


def codigo_jogador_existe(codigo, ignorar_time=None, ignorar_jogador=None):
    for indice_time, jogadores in enumerate(jogadores_por_time):
        for indice_jogador, jogador in enumerate(jogadores):
            if indice_time == ignorar_time and indice_jogador == ignorar_jogador:
                continue
            if jogador[0].casefold() == codigo.casefold():
                return True
    return False


def ler_codigo_jogador(ignorar_time=None, ignorar_jogador=None):
    while True:
        codigo = normalizar_espacos(
            input("Código anonimizado do jogador (ex.: 2C-07): ").strip().upper()
        )
        if not codigo:
            print("ERRO: o código não pode ficar vazio.")
            continue
        if not 2 <= len(codigo) <= 15:
            print("ERRO: o código deve ter entre 2 e 15 caracteres.")
            continue
        if not texto_tem_caracteres_validos(codigo):
            print("ERRO: use apenas letras, números, espaços ou hífen.")
            continue
        if codigo_jogador_existe(codigo, ignorar_time, ignorar_jogador):
            print("ERRO: já existe um jogador com esse código.")
            continue
        return codigo


def ler_posicao():
    posicoes = ["Goleiro", "Fixo", "Ala", "Pivô"]
    print("\nPOSIÇÕES")
    for i, posicao in enumerate(posicoes, start=1):
        print(f"{i} - {posicao}")
    opcao = ler_opcao("Escolha a posição: ", 1, len(posicoes))
    return posicoes[opcao - 1]


def listar_times():
    if not times:
        print("\nNenhum time cadastrado.")
        return
    print("\nTIMES CADASTRADOS")
    print("-" * 55)
    print(f"{'CÓDIGO':<10}{'TIME':<30}{'JOGADORES':>10}")
    print("-" * 55)
    for i, time in enumerate(times):
        print(f"{i:<10}{time:<30}{len(jogadores_por_time[i]):>10}")


def escolher_time(mensagem="Código do time: "):
    while True:
        codigo = ler_inteiro(mensagem)
        if 0 <= codigo < len(times):
            return codigo
        print(f"ERRO: escolha um time entre 0 e {len(times) - 1}.")


def listar_jogadores(indice_time):
    jogadores = jogadores_por_time[indice_time]
    print(f"\nJOGADORES DO TIME {times[indice_time]}")
    print("-" * 65)
    if not jogadores:
        print("Nenhum jogador cadastrado.")
        return
    print(f"{'ID':<5}{'CÓDIGO':<18}{'POSIÇÃO':<15}{'PRESENÇA':<12}")
    print("-" * 65)
    for i, jogador in enumerate(jogadores):
        situacao = "Presente" if jogador[2] else "Ausente"
        print(f"{i:<5}{jogador[0]:<18}{jogador[1]:<15}{situacao:<12}")


def escolher_jogador(indice_time):
    return ler_opcao(
        "Código numérico do jogador: ",
        0,
        len(jogadores_por_time[indice_time]) - 1,
    )


def cadastrar_time():
    if campeonato_iniciado:
        print("ERRO: não é permitido cadastrar times após o início do campeonato.")
        return
    if len(times) >= MAX_TIMES:
        print(f"ERRO: o limite máximo de {MAX_TIMES} times foi atingido.")
        return

    print("\nCADASTRO DE TIME")
    nome = ler_nome_time()
    times.append(nome)
    classificacao.append([0, 0, 0, 0, 0, 0, 0, 0])
    jogadores_por_time.append([])
    for linha in confrontos:
        linha.append("-")
    confrontos.append(["-"] * len(times))
    print(f"Time '{nome}' cadastrado com sucesso.")


def editar_time():
    if not times:
        print("ERRO: não existem times cadastrados.")
        return
    listar_times()
    indice = escolher_time("\nCódigo do time que será editado: ")
    nome_antigo = times[indice]
    print(f"Nome atual: {nome_antigo}")
    times[indice] = ler_nome_time(indice)
    print(f"Time '{nome_antigo}' alterado para '{times[indice]}' com sucesso.")


def remover_time():
    if campeonato_iniciado:
        print("ERRO: não é permitido remover times após o início do campeonato.")
        return
    if not times:
        print("ERRO: não existem times cadastrados.")
        return

    listar_times()
    indice = escolher_time("\nCódigo do time que será removido: ")
    nome = times[indice]
    quantidade = len(jogadores_por_time[indice])
    if quantidade:
        print(f"ATENÇÃO: o time possui {quantidade} jogador(es) cadastrado(s).")
    if confirmar(f"Deseja realmente remover o time '{nome}'?") == "N":
        print("Remoção cancelada.")
        return

    times.pop(indice)
    classificacao.pop(indice)
    jogadores_por_time.pop(indice)
    confrontos.pop(indice)
    for linha in confrontos:
        linha.pop(indice)
    print(f"Time '{nome}' removido com sucesso.")


def cadastrar_jogador():
    if not times:
        print("ERRO: cadastre pelo menos um time antes de cadastrar jogadores.")
        return
    listar_times()
    indice_time = escolher_time("\nCódigo do time do jogador: ")
    print(f"\nCADASTRO DE JOGADOR NO TIME {times[indice_time]}")
    codigo = ler_codigo_jogador()
    posicao = ler_posicao()
    jogadores_por_time[indice_time].append([codigo, posicao, True])
    print(f"Jogador {codigo}, posição {posicao}, cadastrado no time {times[indice_time]}.")


def consultar_jogadores_time():
    if not times:
        print("ERRO: não existem times cadastrados.")
        return
    listar_times()
    indice_time = escolher_time("\nCódigo do time: ")
    listar_jogadores(indice_time)


def editar_jogador():
    if not times:
        print("ERRO: não existem times cadastrados.")
        return
    listar_times()
    indice_time = escolher_time("\nCódigo do time do jogador: ")
    jogadores = jogadores_por_time[indice_time]
    if not jogadores:
        print(f"ERRO: o time {times[indice_time]} não possui jogadores.")
        return

    listar_jogadores(indice_time)
    indice_jogador = escolher_jogador(indice_time)
    jogador = jogadores[indice_jogador]

    if confirmar("Deseja alterar o código do jogador?") == "S":
        jogador[0] = ler_codigo_jogador(indice_time, indice_jogador)
    if confirmar("Deseja alterar a posição do jogador?") == "S":
        jogador[1] = ler_posicao()
    print("Jogador atualizado com sucesso.")


def remover_jogador():
    if not times:
        print("ERRO: não existem times cadastrados.")
        return
    listar_times()
    indice_time = escolher_time("\nCódigo do time do jogador: ")
    jogadores = jogadores_por_time[indice_time]
    if not jogadores:
        print(f"ERRO: o time {times[indice_time]} não possui jogadores.")
        return

    listar_jogadores(indice_time)
    indice_jogador = escolher_jogador(indice_time)
    codigo = jogadores[indice_jogador][0]
    if confirmar(f"Deseja remover o jogador {codigo}?") == "N":
        print("Remoção cancelada.")
        return
    jogadores.pop(indice_jogador)
    print(f"Jogador {codigo} removido com sucesso.")


def alterar_presenca():
    if not times:
        print("ERRO: não existem times cadastrados.")
        return
    listar_times()
    indice_time = escolher_time("\nCódigo do time do jogador: ")
    jogadores = jogadores_por_time[indice_time]
    if not jogadores:
        print(f"ERRO: o time {times[indice_time]} não possui jogadores.")
        return

    listar_jogadores(indice_time)
    indice_jogador = escolher_jogador(indice_time)
    jogador = jogadores[indice_jogador]
    jogador[2] = not jogador[2]
    situacao = "presente" if jogador[2] else "ausente"
    print(f"Presença atualizada: jogador {jogador[0]} está marcado como {situacao}.")


def registrar_partida():
    global campeonato_iniciado

    if len(times) < 2:
        print("ERRO: cadastre pelo menos dois times antes de registrar uma partida.")
        return

    listar_times()
    print("\nSELEÇÃO DOS TIMES")
    time_a = escolher_time("Código do primeiro time: ")
    while True:
        time_b = escolher_time("Código do segundo time: ")
        if time_a != time_b:
            break
        print("ERRO: um time não pode jogar contra ele mesmo.")

    if confrontos[time_a][time_b] != "-":
        print("ERRO: já existe um resultado registrado para esse confronto.")
        print(f"Resultado atual: {times[time_a]} {confrontos[time_a][time_b]} {times[time_b]}")
        return

    print(f"\nPARTIDA: {times[time_a]} x {times[time_b]}")
    gols_a = ler_gols(f"Gols do {times[time_a]}: ")
    gols_b = ler_gols(f"Gols do {times[time_b]}: ")
    if confirmar(
        f"Confirmar resultado: {times[time_a]} {gols_a} x {gols_b} {times[time_b]}?"
    ) == "N":
        print("Registro da partida cancelado.")
        return

    atualizar_classificacao(time_a, time_b, gols_a, gols_b)
    campeonato_iniciado = True
    print("Resultado registrado com sucesso.")


def atualizar_classificacao(a, b, gols_a, gols_b):
    classificacao[a][0] += 1
    classificacao[b][0] += 1
    classificacao[a][4] += gols_a
    classificacao[b][4] += gols_b
    classificacao[a][5] += gols_b
    classificacao[b][5] += gols_a

    if gols_a > gols_b:
        classificacao[a][1] += 1
        classificacao[a][7] += 3
        classificacao[b][3] += 1
    elif gols_a == gols_b:
        classificacao[a][2] += 1
        classificacao[b][2] += 1
        classificacao[a][7] += 1
        classificacao[b][7] += 1
    else:
        classificacao[b][1] += 1
        classificacao[b][7] += 3
        classificacao[a][3] += 1

    classificacao[a][6] = classificacao[a][4] - classificacao[a][5]
    classificacao[b][6] = classificacao[b][4] - classificacao[b][5]
    confrontos[a][b] = f"{gols_a}x{gols_b}"
    confrontos[b][a] = f"{gols_b}x{gols_a}"


def mostrar_classificacao():
    if not times:
        print("ERRO: não existem times cadastrados.")
        return

    ranking = [[times[i]] + classificacao[i].copy() for i in range(len(times))]
    ranking.sort(
        key=lambda linha: (
            -linha[8],
            -linha[2],
            -linha[7],
            -linha[5],
            linha[0].casefold(),
        )
    )

    print("\nCLASSIFICAÇÃO")
    print("-" * 85)
    print(
        f"{'POS':<5}{'TIME':<22}{'PTS':>5}{'PJ':>5}{'V':>5}"
        f"{'E':>5}{'D':>5}{'GP':>5}{'GC':>5}{'SG':>5}"
    )
    print("-" * 85)

    for posicao, linha in enumerate(ranking, start=1):
        nome, pj, v, e, d, gp, gc, sg, pts = linha
        print(
            f"{posicao:<5}{nome[:20]:<22}{pts:>5}{pj:>5}{v:>5}"
            f"{e:>5}{d:>5}{gp:>5}{gc:>5}{sg:>5}"
        )
    print("-" * 85)
    print("Critérios: pontos, vitórias, saldo de gols, gols pró e nome.")


def mostrar_confrontos():
    if not times:
        print("ERRO: não existem times cadastrados.")
        return

    largura_coluna = 10
    largura_nome = 16
    print("\nMATRIZ DE CONFRONTOS")
    print("Cada célula mostra o resultado do time da linha contra o time da coluna.\n")
    print(" " * largura_nome, end="")
    for time in times:
        print(f"{time[:largura_coluna - 1]:>{largura_coluna}}", end="")
    print()

    for i, time in enumerate(times):
        print(f"{time[:largura_nome - 1]:<{largura_nome}}", end="")
        for j in range(len(times)):
            valor = "X" if i == j else confrontos[i][j]
            print(f"{valor:>{largura_coluna}}", end="")
        print()


def mostrar_menu():
    print("\n" + "=" * 65)
    print("ESCALA2C - CAMPEONATO INTERCLASSE")
    print("=" * 65)
    print(" 1 - Cadastrar time")
    print(" 2 - Editar time")
    print(" 3 - Remover time")
    print("-" * 65)
    print(" 4 - Cadastrar jogador")
    print(" 5 - Listar jogadores de um time")
    print(" 6 - Editar jogador")
    print(" 7 - Remover jogador")
    print(" 8 - Alterar presença de jogador")
    print("-" * 65)
    print(" 9 - Registrar partida")
    print("10 - Mostrar classificação")
    print("11 - Mostrar confrontos")
    print("-" * 65)
    print(" 0 - Sair")
    print("=" * 65)


def executar_programa():
    while True:
        mostrar_menu()
        opcao = ler_opcao("Escolha uma opção: ", 0, 11)

        if opcao == 1:
            cadastrar_time()
        elif opcao == 2:
            editar_time()
        elif opcao == 3:
            remover_time()
        elif opcao == 4:
            cadastrar_jogador()
        elif opcao == 5:
            consultar_jogadores_time()
        elif opcao == 6:
            editar_jogador()
        elif opcao == 7:
            remover_jogador()
        elif opcao == 8:
            alterar_presenca()
        elif opcao == 9:
            registrar_partida()
        elif opcao == 10:
            mostrar_classificacao()
        elif opcao == 11:
            mostrar_confrontos()
        elif confirmar("Deseja realmente encerrar o sistema?") == "S":
            print("\nSistema Escala2C encerrado.")
            break


if __name__ == "__main__":
    executar_programa()
