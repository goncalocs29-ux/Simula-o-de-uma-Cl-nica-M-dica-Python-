# Simula-o-de-uma-Cl-nica-M-dica-Python-
Projeto final feito em python 

import random
import numpy as np
import matplotlib.pyplot as plt
import json
import FreeSimpleGUI as sg

# ========== CONFIGURAÇÕES INICIAIS ==========
sg.theme('DarkBlue3')

# Parâmetros padrão
TEMPO_SIMULACAO = 8 * 60  # 8 horas em minutos
INICIO_BALCAO = "inicio_balcao"
FIM_BALCAO = "fim_balcao"
CHEGADA = "chegada"
SAIDA = "saída"

# Sistema de urgências
URGENCIAS = {"emergente": 0,
            "urgente": 1, 
            "pouco_urgente": 2, 
            "inapropriado": 3, }

#probablidade de cada urgência

pro_urg=[0.1, 0.3, 0.4, 0.2]

# ========== FUNÇÕES DO SISTEMA ==========
def carregar_dados_pacientes():

    with open("pessoas.json", "r", encoding="utf-8") as doc:
        dados_pacientes = json.load(doc)
    pessoas = []
    for paciente in dados_pacientes:
        pessoas.append((paciente["nome"], paciente["idade"], paciente["sexo"]))
        
    random.shuffle(pessoas)
    return pessoas


def taxa_chegada(t, taxas):
    if t < 60:          # primeira onda (0-60 min)
        return taxas[0] / 60
    elif t < 180:       # segunda onda (60-180 min)
        return taxas[1] / 60
    elif t < 300:       # terceira onda (180-300 min)
        return taxas[2] / 60
    elif t < 480:       # quarta onda (300-480 min)
        return taxas[3] / 60
    else:               # fim das ondas de chegada
        return 0

#Atribui uma classificação de urgência com base nas probabilidades de cada tipo de URGENCIAS
def atribuir_urgencia():
    tipo=list(URGENCIAS.keys())
    escolha=np.random.choice(tipo,p=pro_urg)
    codigo=URGENCIAS[escolha]
    return escolha, codigo

# Modelo para o evento
def e_tempo(e):
    return e[0]

def e_tipo(e):
    return e[1]

def e_doente(e):
    return e[2]

def e_urgencia(e):
    return e[3]

# Funções para a Queue de Eventos
def procuraPosQueue(q, t):
    i = 0
    while i < len(q) and t > q[i][0]:
        i = i + 1
    return i

def enqueue(q, e):
    pos = procuraPosQueue(q, e[0])
    return q[:pos] + [e] + q[pos:]

def dequeue(q):
    if not q:
        return None, q
    e = q[0]
    q = q[1:]
    return e, q

# Modelo para balcão
def b_ocupado(b):
    return b[1]

def bOcupa(b):
    b[1] = not b[1]
    return b

def b_doente_corrente(b):
    return b[2]

def bDoenteCorrente(b, d):
    b[2] = d
    return b

def procuraBalcao(lista):
    res = None
    for b in lista:
        if not b[1]:  
            res = b
            break 
    return res

# Modelo para o médico
def m_id(e):
    return e[0]

def m_ocupado(e):
    return e[1]

def mOcupa(m):
    m[1] = not m[1]
    return m

def m_doente_corrente(e):
    return e[2]

def mDoenteCorrente(m, d):
    m[2] = d
    return m

def m_total_tempo_ocupado(e):
    return e[3]

def mTempoOcupado(m, t):
    m[3] = t
    return m

def m_inicio_ultima_consulta(e):
    return e[4]

def mInicioConsulta(m, t):
    m[4] = t
    return m

# Funções para gerenciar fila prioritária
def insere_na_fila_prioritaria(fila, doente_id, tempo_chegada, urgencia_codigo):
    
    elemento = (urgencia_codigo, tempo_chegada, doente_id)
    pos = 0
    while pos < len(fila) and fila[pos][0] <= urgencia_codigo:
        if fila[pos][0] == urgencia_codigo and fila[pos][1] <= tempo_chegada:
            pos += 1
        elif fila[pos][0] < urgencia_codigo:
            pos += 1
        else:
            break

    fila.insert(pos, elemento)
    return fila

def remove_da_fila_prioritaria(fila):
    if not fila:
        res = None
    else:
        res=fila.pop(0)
    return res, fila

# Funções para gerar tempos
def gera_intervalo_tempo_chegada(lmbda):
    return np.random.exponential(1 / lmbda)


def gera_tempo_aleatório(tempo_medio, distribuicao):
    if distribuicao == "exponential":
        return np.random.exponential(tempo_medio)
    elif distribuicao == "normal":
        return max(0.1, np.random.normal(tempo_medio, tempo_medio/2))
    elif distribuicao == "uniform":
        return np.random.uniform(tempo_medio * 0.5, tempo_medio * 1.5)
    return np.random.exponential(tempo_medio)

# Procura o primeiro médico livre
def procuraMedico(lista):
    res = None
    i = 0
    encontrado = False
    while not encontrado and i < len(lista):
        if not lista[i][1]:
            res = lista[i]
            encontrado = True
        i = i + 1
    return res

def registrar_estado(tempo, fila_balcao, fila_consulta, medicos, tempo_history, fila_balcao_history, fila_consulta_history, ocupacao_history, taxa_chegada_history, taxas):
    tempo_history.append(tempo)
    fila_balcao_history.append(len(fila_balcao))
    fila_consulta_history.append(len(fila_consulta))
    
    ocupados = sum([1 for m in medicos if m_ocupado(m)])
    ocupacao_percent = (ocupados / len(medicos)) * 100 if medicos else 0
    ocupacao_history.append(ocupacao_percent)
    
    # Registrar taxa de chegada atual (em pacientes/hora)
    taxa_atual = taxa_chegada(tempo, taxas) * 60  # Converter para pacientes/hora
    taxa_chegada_history.append(taxa_atual)

def criar_graficos(tempo_history, fila_balcao_history, fila_consulta_history, ocupacao_history, taxa_chegada_history, taxas_config):
    
    # Gráfico 1: Evolução das filas e ocupação
    plt.figure(figsize=(15, 5))
    
    # Subplot 1: Fila do balcão
    plt.subplot(1, 3, 1)
    plt.plot(tempo_history, fila_balcao_history, 'b-', linewidth=1, label='Fila do Balcão')
    plt.fill_between(tempo_history, 0, fila_balcao_history, alpha=0.3, color='blue')
    plt.xlabel('Tempo (minutos)')
    plt.ylabel('Tamanho da Fila')
    plt.title('Evolução da Fila do Balcão')
    plt.grid(True, alpha=0.3)
    plt.axvline(x=480, color='gray', linestyle='--', alpha=0.5, label='Fim das 8h de simulação')
    plt.axvspan(xmin=480, xmax=max(tempo_history), alpha=0.05, color='red')
    plt.legend()
    
    # Subplot 2: Fila de consulta
    plt.subplot(1, 3, 2)
    plt.plot(tempo_history, fila_consulta_history, 'r-', linewidth=1, label='Fila de Consulta')
    plt.fill_between(tempo_history, 0, fila_consulta_history, alpha=0.3, color='red')
    plt.xlabel('Tempo (minutos)')
    plt.ylabel('Tamanho da Fila')
    plt.title('Evolução da Fila de Consulta')
    plt.grid(True, alpha=0.3)
    plt.axvline(x=480, color='gray', linestyle='--', alpha=0.5, label='Fim das 8h de simulação')
    plt.axvspan(xmin=480, xmax=max(tempo_history), alpha=0.05, color='red')
    plt.legend()
    
    # Subplot 3: Ocupação dos médicos
    plt.subplot(1, 3, 3)
    plt.plot(tempo_history, ocupacao_history, 'g-', linewidth=1, label='Ocupação')
    plt.fill_between(tempo_history, 0, ocupacao_history, alpha=0.3, color='green')
    plt.xlabel('Tempo (minutos)')
    plt.ylabel('Ocupação (%)')
    plt.title('Evolução da Ocupação dos Médicos')
    plt.grid(True, alpha=0.3)
    plt.ylim(0, 105)
    plt.axvline(x=480, color='gray', linestyle='--', alpha=0.5, label='Fim das 8h de simulação')
    plt.axvspan(xmin=480, xmax=max(tempo_history), alpha=0.05, color='red')
    plt.legend()
    
    plt.tight_layout()
    plt.savefig('evolucao_filas_ocupacao.png', dpi=150, bbox_inches='tight')
    plt.show()
    
    # Gráfico 2: Comparação das duas filas
    plt.figure(figsize=(14, 6))
    plt.plot(tempo_history, fila_balcao_history, 'b-', linewidth=2, label='Fila do Balcão')
    plt.plot(tempo_history, fila_consulta_history, 'r-', linewidth=2, label='Fila de Consulta')
    plt.fill_between(tempo_history, 0, fila_balcao_history, alpha=0.2, color='blue')
    plt.fill_between(tempo_history, 0, fila_consulta_history, alpha=0.2, color='red')
    plt.xlabel('Tempo (minutos)')
    plt.ylabel('Tamanho da Fila')
    plt.title('Comparação das Filas do Hospital')
    plt.grid(True, alpha=0.3)
    plt.axvline(x=480, color='gray', linestyle='--', alpha=0.5, label='Fim das 8h de simulação')
    plt.axvspan(xmin=480, xmax=max(tempo_history), alpha=0.05, color='red')
    plt.legend()
    plt.savefig('comparacao_filas.png', dpi=150, bbox_inches='tight') 
    plt.show()
    
    # Gráfico 3: Relação entre tamanho médio da fila e taxa de chegada
    if len(tempo_history) > 1 and len(taxa_chegada_history) > 0:
        
        # Calcular tamanho médio das filas por faixa de taxa de chegada
        taxa_min = min(taxa_chegada_history)
        taxa_max = max(taxa_chegada_history)
        
        if taxa_max > taxa_min:
            num_bins = 10
            bins = np.linspace(taxa_min, taxa_max, num_bins + 1)
            
            # Agrupar dados por faixa de taxa
            fila_balcao_por_taxa = []
            fila_consulta_por_taxa = []
            taxas_centro = []
            
            for i in range(num_bins):
                taxa_inicio = bins[i]
                taxa_fim = bins[i+1]
                indices = [idx for idx, taxa in enumerate(taxa_chegada_history) 
                          if taxa_inicio <= taxa < taxa_fim]
                
                if indices:
                    media_balcao = np.mean([fila_balcao_history[idx] for idx in indices])
                    media_consulta = np.mean([fila_consulta_history[idx] for idx in indices])
                    taxa_media = (taxa_inicio + taxa_fim) / 2
                    
                    fila_balcao_por_taxa.append(media_balcao)
                    fila_consulta_por_taxa.append(media_consulta)
                    taxas_centro.append(taxa_media)
            
            if taxas_centro:
                plt.figure(figsize=(12, 6))
                
                plt.subplot(1, 2, 1)
                plt.scatter(taxas_centro, fila_balcao_por_taxa, c='blue', s=100, alpha=0.7)
                plt.plot(taxas_centro, fila_balcao_por_taxa, 'b-', alpha=0.5)
                plt.xlabel('Taxa de Chegada (pacientes/hora)')
                plt.ylabel('Tamanho Médio da Fila do Balcão')
                plt.title('Relação: Taxa de Chegada vs Fila do Balcão')
                plt.grid(True, alpha=0.3)
                
                # Adicionar linha de regressão linear para fila balcão
                if len(taxas_centro) > 1:
                    z = np.polyfit(taxas_centro, fila_balcao_por_taxa, 1)
                    p = np.poly1d(z)
                    plt.plot(taxas_centro, p(taxas_centro), "b--", alpha=0.8, label='Tendência')
                    plt.legend()
                
                plt.subplot(1, 2, 2)
                plt.scatter(taxas_centro, fila_consulta_por_taxa, c='red', s=100, alpha=0.7)
                plt.plot(taxas_centro, fila_consulta_por_taxa, 'r-', alpha=0.5)
                plt.xlabel('Taxa de Chegada (pacientes/hora)')
                plt.ylabel('Tamanho Médio da Fila de Consulta')
                plt.title('Relação: Taxa de Chegada vs Fila de Consulta')
                plt.grid(True, alpha=0.3)
                
                # Adicionar linha de regressão linear para fila consulta
                if len(taxas_centro) > 1:
                    z = np.polyfit(taxas_centro, fila_consulta_por_taxa, 1)
                    p = np.poly1d(z)
                    plt.plot(taxas_centro, p(taxas_centro), "r--", alpha=0.8, label='Tendência')
                    plt.legend()
                
                plt.tight_layout()
                plt.savefig('relacao_taxa_filas.png', dpi=150, bbox_inches='tight')
                plt.show()
        
        # Gráfico 4: Taxa de chegada ao longo do tempo
        plt.figure(figsize=(12, 6))
        plt.plot(tempo_history, taxa_chegada_history, 'purple', linewidth=2, label='Taxa de Chegada')
        plt.fill_between(tempo_history, 0, taxa_chegada_history, alpha=0.3, color='purple')
        plt.xlabel('Tempo (minutos)')
        plt.ylabel('Taxa de Chegada (pacientes/hora)')
        plt.title('Evolução da Taxa de Chegada')
        plt.grid(True, alpha=0.3)
        
        # Adicionar linhas verticais para mostrar as mudanças de onda
        plt.axvline(x=60, color='gray', linestyle='--', alpha=0.5, label='Mudança de onda')
        plt.axvline(x=180, color='gray', linestyle='--', alpha=0.5)
        plt.axvline(x=300, color='gray', linestyle='--', alpha=0.5)
        plt.axvline(x=480, color='gray', linestyle='--', alpha=0.5)
        plt.axvspan(xmin=480, xmax=max(tempo_history), alpha=0.05, color='red', label='Fim das 8h de simulação')
        
        plt.legend()
        plt.savefig('evolucao_taxa_chegada.png', dpi=150, bbox_inches='tight')
        plt.show()

def simula(params, output_element=None):
    # Extrair parâmetros
    taxa_1 = params['taxa_1']
    taxa_2 = params['taxa_2']
    taxa_3 = params['taxa_3']
    taxa_4 = params['taxa_4']
    taxas = [taxa_1, taxa_2, taxa_3, taxa_4]  # Lista de taxas por onda
    
    num_balcoes = params['num_balcoes']
    tempo_medio_balcao = params['tempo_medio_balcao']
    num_medicos = params['num_medicos']
    tempo_medio_consulta = params['tempo_medio_consulta']
    distribuicao_consulta = params['distribuicao_consulta']
    distribuicao_balcao = params['distribuicao_balcao']
    
    # Carregar dados dos pacientes
    pessoas = carregar_dados_pacientes()
    if not pessoas:
        if output_element:
            output_element.update("Erro: Não foi possível carregar os dados dos pacientes.\n", append=True)
        return None, None, None, None, None, None, None
    
    # Inicialização
    tempo_atual = 0.0
    contadorDoentes = 1
    indice_pessoa = 0
    
    # Estruturas de dados
    queueEventos = []
    fila_balcao = []
    fila_consulta = []
    
    chegadas = {}
    urgencias = {}
    pacientes_info = {}
    
    
    tempo_inicio_balcao = {}  
    tempo_fim_balcao = {}     
    tempo_inicio_consulta = {} 
    tempo_fim_consulta = {}  
    
    
    medicos = [[f"médico {i+1}", False, None, 0.0, 0.0] for i in range(num_medicos)]
    balcoes = [[f"balcão {i+1}", False, None] for i in range(num_balcoes)]
    
    
    tempo_history = []
    fila_balcao_history = []
    fila_consulta_history = []
    ocupacao_history = []
    taxa_chegada_history = []
    
    
    contagem_sexo = {"masculino": 0, "feminino": 0, "outro": 0}
    
    # --- Geração das chegadas de doentes
    # Começar com a primeira taxa de chegada
    TAXA_CHEGADA = taxa_chegada(tempo_atual, taxas)
    tempo_atual = tempo_atual + gera_intervalo_tempo_chegada(TAXA_CHEGADA)
    
    while tempo_atual < TEMPO_SIMULACAO:
        doente_id = "d" + str(contadorDoentes)
        contadorDoentes += 1
     
        # Obter informações da pessoa
        nome, idade, sexo = pessoas[indice_pessoa]
        indice_pessoa = (indice_pessoa + 1) % len(pessoas)
        
        # Atribuir urgência
        tipo_urgencia, codigo_urgencia = atribuir_urgencia()
        
        # Armazenar informações do doente
        chegadas[doente_id] = tempo_atual
        urgencias[doente_id] = (tipo_urgencia, codigo_urgencia)
        pacientes_info[doente_id] = {
            "nome": nome,
            "idade": idade,
            "sexo": sexo,
            "tipo_urgencia": tipo_urgencia,
            "codigo_urgencia": codigo_urgencia,
            "tempo_chegada": tempo_atual
        }
        
        # Criar evento de chegada
        queueEventos = enqueue(queueEventos, (tempo_atual, CHEGADA, doente_id, codigo_urgencia))
        
        # Atualizar taxa de chegada baseada no tempo atual
        TAXA_CHEGADA = taxa_chegada(tempo_atual, taxas)
        intervalo = gera_intervalo_tempo_chegada(TAXA_CHEGADA)
        tempo_atual = tempo_atual + intervalo
    
    # --- Tratamento dos eventos
    doentes_atendidos = 0
    tempo_atual = 0.0

    if output_element:
        output_element.update("\n=== INÍCIO DA SIMULAÇÃO ===\n", append=True)
        output_element.update(f"Parâmetros:\n", append=True)
        output_element.update(f"  Taxa 1 (0-60 min): {taxa_1} pacientes/hora\n", append=True)
        output_element.update(f"  Taxa 2 (60-180 min): {taxa_2} pacientes/hora\n", append=True)
        output_element.update(f"  Taxa 3 (180-300 min): {taxa_3} pacientes/hora\n", append=True)
        output_element.update(f"  Taxa 4 (300-480 min): {taxa_4} pacientes/hora\n", append=True)
        output_element.update(f"  Balcões: {num_balcoes}\n", append=True)
        output_element.update(f"  Médicos: {num_medicos}\n", append=True)
        output_element.update(f"  Tempo médio consulta: {tempo_medio_consulta} min\n", append=True)
        output_element.update(f"  Tempo médio atendimento balcão: {tempo_medio_balcao} min\n", append=True)
        output_element.update(f"Distribuição de urgências: Emergente({pro_urg[0]*100}%), "
                           f"Urgente({pro_urg[1]*100}%), "
                           f"Pouco Urgente({pro_urg[2]*100}%), "
                           f"Inapropriado({pro_urg[3]*100}%)\n\n", append=True)

    while queueEventos != []:
        evento, queueEventos = dequeue(queueEventos)
        tempo_atual = e_tempo(evento)
        doente_id = e_doente(evento)
        codigo_urgencia = e_urgencia(evento)
        
     
        paciente = pacientes_info[doente_id]
        tipo_urgencia = paciente["tipo_urgencia"]
        nome = paciente["nome"]
        idade = paciente["idade"]
        sexo = paciente["sexo"]
       
        if output_element:
            output_element.update(f"[Tempo: {tempo_atual:.1f}] {e_tipo(evento)} - {nome} ({idade} anos, {sexo}), "
                              f"Urgência: {tipo_urgencia} \n", append=True)

        if e_tipo(evento) == CHEGADA:
            balcao_livre = procuraBalcao(balcoes)
                
                # Registrar início do atendimento no balcão
            if balcao_livre:

                tempo_inicio_balcao[doente_id] = tempo_atual
                balcao_livre = bOcupa(balcao_livre)
                balcao_livre = bDoenteCorrente(balcao_livre, doente_id)
                tempo_balcao = gera_tempo_aleatório(tempo_medio_balcao, distribuicao_balcao)
                queueEventos = enqueue(queueEventos, (tempo_atual + tempo_balcao, FIM_BALCAO, doente_id, codigo_urgencia))
            else:
                fila_balcao = insere_na_fila_prioritaria(fila_balcao, doente_id, tempo_atual, codigo_urgencia)
                if output_element:
                    output_element.update(f"  -> Fila do balcão ({len(fila_balcao)}): {[pacientes_info[d[2]]['nome'][:10] for d in fila_balcao]}\n", append=True)

        elif e_tipo(evento) == FIM_BALCAO:
            # Registrar fim do atendimento no balcão
            tempo_fim_balcao[doente_id] = tempo_atual
        
            # Libertar balcão
            for b in balcoes:
                if b_doente_corrente(b) == doente_id:
                    b = bOcupa(b)
                    b = bDoenteCorrente(b, None)
                    break

            # Chamar próximo da fila do balcão
            if fila_balcao:
                prox_urgencia, prox_tempo, prox_id = fila_balcao.pop(0)
                b_livre = procuraBalcao(balcoes)
                if b_livre:
                    # Registrar início do atendimento no balcão para próximo paciente
                    tempo_inicio_balcao[prox_id] = tempo_atual
                    b_livre = bOcupa(b_livre)
                    b_livre = bDoenteCorrente(b_livre, prox_id)
                    tempo_balcao = gera_tempo_aleatório(tempo_medio_balcao, distribuicao_balcao)
                    queueEventos = enqueue(queueEventos, (tempo_atual + tempo_balcao, FIM_BALCAO, prox_id, prox_urgencia))
                    if output_element:
                        output_element.update(f"  -> Próximo no balcão: {pacientes_info[prox_id]['nome'][:15]}\n", append=True)

            # O doente segue para a consulta
            medico_livre = procuraMedico(medicos)
            if medico_livre:
                # registrar início da consulta
                tempo_inicio_consulta[doente_id] = tempo_atual
                medico_livre = mOcupa(medico_livre)
                medico_livre = mInicioConsulta(medico_livre, tempo_atual)
                medico_livre = mDoenteCorrente(medico_livre, doente_id)
                tempo_consulta = gera_tempo_aleatório(tempo_medio_consulta, distribuicao_consulta)
                queueEventos = enqueue(queueEventos, (tempo_atual + tempo_consulta, SAIDA, doente_id, codigo_urgencia))
                if output_element:
                    output_element.update(f"  -> Consulta iniciada para {nome[:15]} com médico {m_id(medico_livre)}\n", append=True)
            else:
                # Adicionar à fila prioritária de consulta
                tempo_chegada = chegadas[doente_id]
                fila_consulta = insere_na_fila_prioritaria(fila_consulta, doente_id, tempo_chegada, codigo_urgencia)
                if output_element:
                    output_element.update(f"  -> Fila de consulta ({len(fila_consulta)}): {[pacientes_info[d[2]]['nome'][:10] for d in fila_consulta]}\n", append=True)

        elif e_tipo(evento) == SAIDA:
            doentes_atendidos += 1
            
            # Registrar fim da consulta
            tempo_fim_consulta[doente_id] = tempo_atual
            
            # Contar sexo do paciente atendido
            if doente_id in pacientes_info:
                sexo_paciente = pacientes_info[doente_id]["sexo"]
                if sexo_paciente in contagem_sexo:
                    contagem_sexo[sexo_paciente] += 1
            
            # Libertar o médico
            i = 0
            encontrado = False
            while i < len(medicos) and not encontrado:
                if m_doente_corrente(medicos[i]) == doente_id:
                    medicos[i] = mOcupa(medicos[i])
                    medicos[i] = mDoenteCorrente(medicos[i], None)
                    tempo_consulta_real = tempo_atual - m_inicio_ultima_consulta(medicos[i])
                    medicos[i] = mTempoOcupado(medicos[i], m_total_tempo_ocupado(medicos[i]) + tempo_consulta_real)
                    encontrado = True
                i = i + 1

            # Atender próximo da fila prioritária
            if fila_consulta:
                proximo, fila_consulta = remove_da_fila_prioritaria(fila_consulta)
                urgencia_codigo, tempo_chegada, prox_id = proximo
                
                medico_livre = procuraMedico(medicos)
                if medico_livre:
                    # Registrar início da consulta para próximo paciente
                    tempo_inicio_consulta[prox_id] = tempo_atual
                    medico_livre = mOcupa(medico_livre)
                    medico_livre = mInicioConsulta(medico_livre, tempo_atual)
                    medico_livre = mDoenteCorrente(medico_livre, prox_id)
                    tempo_consulta = gera_tempo_aleatório(tempo_medio_consulta, distribuicao_consulta)
                    
                    prox_nome = pacientes_info[prox_id]["nome"]
                    prox_urgencia_tipo = pacientes_info[prox_id]["tipo_urgencia"]
                    
                    queueEventos = enqueue(queueEventos, (tempo_atual + tempo_consulta, SAIDA, prox_id, urgencia_codigo))
                    if output_element:
                        output_element.update(f"  -> Próximo da fila: {prox_nome[:15]} (Urgência: {prox_urgencia_tipo})\n", append=True)
        
        registrar_estado(tempo_atual, fila_balcao, fila_consulta, medicos,
                         tempo_history, fila_balcao_history, fila_consulta_history, 
                         ocupacao_history, taxa_chegada_history, taxas)

    # --- Estatísticas finais
    # Calcular tempos de espera e tempos totais
    tempos_espera_balcao = []
    tempos_espera_consulta = []
    tempos_totais_hospital = []

    for doente_id in pacientes_info:
        if doente_id in tempo_fim_consulta:
            tempo_chegada = pacientes_info[doente_id]["tempo_chegada"]
            tempo_saida = tempo_fim_consulta[doente_id]

            # Tempo total no hospital
            tempo_total = tempo_saida - tempo_chegada
            tempos_totais_hospital.append(tempo_total)
            
            # Tempo de espera no balcão
            if doente_id in tempo_inicio_balcao:
                tempo_espera_balcao = tempo_inicio_balcao[doente_id] - tempo_chegada
                tempos_espera_balcao.append(tempo_espera_balcao)
            
            # Tempo de espera na consulta
            if doente_id in tempo_inicio_consulta and doente_id in tempo_fim_balcao:
                tempo_espera_consulta = tempo_inicio_consulta[doente_id] - tempo_fim_balcao[doente_id]
                tempos_espera_consulta.append(tempo_espera_consulta)
    
    # Calcular médias
    media_espera_balcao = np.mean(tempos_espera_balcao) if tempos_espera_balcao else 0
    media_espera_consulta = np.mean(tempos_espera_consulta) if tempos_espera_consulta else 0
    media_tempo_total = np.mean(tempos_totais_hospital)
    # calcular tempo decorrido após o término da simulação
    tempo_decorrido = tempo_atual - TEMPO_SIMULACAO
    
    if output_element:
        output_element.update(f"\n=== FIM DA SIMULAÇÃO ===\n", append=True)
        output_element.update(f"Doentes atendidos: {doentes_atendidos}\n", append=True)
        
        # Estatísticas por tipo de urgência
        output_element.update("\nEstatísticas por tipo de urgência:\n", append=True)
        contadores_urgencia = {"emergente": 0, "urgente": 0, "pouco_urgente": 0, "inapropriado": 0}
        
        for doente_id, info in pacientes_info.items():
            if int(doente_id[1:]) <= doentes_atendidos:
                contadores_urgencia[info["tipo_urgencia"]] += 1
        
        for tipo, dados in URGENCIAS.items():
            count = contadores_urgencia[tipo]
            percentual = (count / doentes_atendidos * 100) if doentes_atendidos > 0 else 0
            output_element.update(f"  {tipo.capitalize()}: {count} doentes ({percentual:.1f}%)\n", append=True)
        
        # Tempo de ocupação dos médicos
        output_element.update("\nTempo de ocupação dos médicos:\n", append=True)
        for i, medico in enumerate(medicos):
            tempo_ocupado = m_total_tempo_ocupado(medico)
            percentual_ocupacao = (tempo_ocupado / TEMPO_SIMULACAO * 100) if TEMPO_SIMULACAO > 0 else 0
            output_element.update(f"  Médico {i+1}: {tempo_ocupado:.1f} min ({percentual_ocupacao:.1f}% de ocupação)\n", append=True)
        percentual_media_ocupacao = (np.mean([m_total_tempo_ocupado(m) / TEMPO_SIMULACAO * 100 for m in medicos])) if medicos else 0
        output_element.update(f"  Percentual médio de ocupação dos médicos: {percentual_media_ocupacao:.1f}%\n", append=True)
        
        # Estatísticas demográficas dos pacientes atendidos
        output_element.update("\nEstatísticas demográficas dos pacientes atendidos:\n", append=True)
        if doentes_atendidos > 0:
            # Usar a contagem de sexo
            total_sexos = sum(contagem_sexo.values())
            if total_sexos > 0:
                percent_m = (contagem_sexo["masculino"] / total_sexos) * 100
                percent_f = (contagem_sexo["feminino"] / total_sexos) * 100
                percent_o = (contagem_sexo["outro"] / total_sexos) * 100
                
                output_element.update(f"  Total pacientes: {total_sexos}\n", append=True)
                output_element.update(f"  Sexo masculino: {contagem_sexo['masculino']} ({percent_m:.1f}%)\n", append=True)
                output_element.update(f"  Sexo feminino: {contagem_sexo['feminino']} ({percent_f:.1f}%)\n", append=True)
                output_element.update(f"  Sexo outro: {contagem_sexo['outro']} ({percent_o:.1f}%)\n", append=True)

        
        # Estatísticas de tempo de espera
        output_element.update("\nEstatísticas de Tempo de Espera:\n", append=True)
        output_element.update(f"  Tempo médio de espera no balcão: {media_espera_balcao:.2f} minutos\n", append=True)
        output_element.update(f"  Tempo médio de espera para consulta: {media_espera_consulta:.2f} minutos\n", append=True)
        output_element.update(f"  Tempo médio total no hospital: {media_tempo_total:.2f} minutos\n", append=True)
        output_element.update(f"  Tempo decorrido após o término da simulação de chegadas: {tempo_decorrido:.2f} minutos\n", append=True)
        if tempos_espera_balcao:
            output_element.update(f"  Máximo tempo de espera no balcão: {max(tempos_espera_balcao):.2f} minutos\n", append=True)
        if tempos_espera_consulta:
            output_element.update(f"  Máximo tempo de espera para consulta: {max(tempos_espera_consulta):.2f} minutos\n", append=True)
        if tempos_totais_hospital:
            output_element.update(f"  Máximo tempo total no hospital: {max(tempos_totais_hospital):.2f} minutos\n", append=True)
        
        # Estatísticas de tamanho médio das filas
        if fila_balcao_history:
            media_fila_balcao = np.mean(fila_balcao_history)
            output_element.update(f"\nTamanho médio da fila do balcão: {media_fila_balcao:.2f} pacientes\n", append=True)
        
        if fila_consulta_history:
            media_fila_consulta = np.mean(fila_consulta_history)
            output_element.update(f"Tamanho médio da fila de consulta: {media_fila_consulta:.2f} pacientes\n", append=True)
        
    
    # Criar gráficos
    criar_graficos(tempo_history, fila_balcao_history, fila_consulta_history, 
                  ocupacao_history, taxa_chegada_history, taxas)
    
    return 

def criar_interface():
    
    layout = [
        [sg.Text("Configuração da Simulação Hospitalar", font=("Helvetica", 16), justification='center')],
        [sg.HorizontalSeparator()],
        [sg.Text("Taxas de Chegada de Pacientes (pacientes por hora)", font=("Helvetica", 12))],
        [sg.Text("Onda 1 (0-60 min):")],
        [sg.Slider(range=(10, 30), default_value=30, orientation='h', size=(30, 15), key='-TAXA1-')],
        [sg.Text("Onda 2 (60-180 min):")],
        [sg.Slider(range=(10, 30), default_value=12, orientation='h', size=(30, 15), key='-TAXA2-')],
        [sg.Text("Onda 3 (180-300 min):")],
        [sg.Slider(range=(10, 30), default_value=15, orientation='h', size=(30, 15), key='-TAXA3-')],
        [sg.Text("Onda 4 (300-480 min):")],
        [sg.Slider(range=(10, 30), default_value=10, orientation='h', size=(30, 15), key='-TAXA4-')],
        [sg.HorizontalSeparator()],
        [sg.Text("Configuração dos Balcões", font=("Helvetica", 12))],
        
        [sg.Text("Número de Balcões:"), sg.Slider(range=(1, 5), default_value=2,orientation='h', size=(20, 15), key='-NUM_BALCOES-')],
        
        [sg.Text("Tempo Médio no Balcão (min):"), sg.Slider(range=(1, 15), default_value=5,orientation='h', size=(20, 15), key='-TEMPO_BALCAO-')],
        
        [sg.Text("Distribuição do Tempo no Balcão:"), sg.Combo(['exponential', 'normal', 'uniform'], default_value='exponential', key='-DIST_BALCAO-', size=(15, 1))],
        
        [sg.HorizontalSeparator()],
        [sg.Text("Configuração das Consultas Médicas", font=("Helvetica", 12))],
        
        [sg.Text("Número de Médicos:"), sg.Slider(range=(1, 5), default_value=3, orientation='h', size=(20, 15), key='-NUM_MEDICOS-')],
        
        [sg.Text("Tempo Médio de Consulta (min):"), sg.Slider(range=(5, 30), default_value=10, orientation='h', size=(20, 15), key='-TEMPO_CONSULTA-')],
       
        [sg.Text("Distribuição do Tempo de Consulta:"), sg.Combo(['exponential', 'normal', 'uniform'], default_value='exponential', key='-DIST_CONSULTA-', size=(15, 1))],
        
        [sg.HorizontalSeparator()],
        
        [sg.Button('Iniciar Simulação', size=(15, 2)), sg.Button('Valores Padrão', size=(15, 2)), sg.Button('Sair', size=(15, 2))],
        
        [sg.Text("", size=(50, 1), key='-STATUS-', text_color='yellow')]
    ]
    
    return sg.Window('Simulação Hospitalar', layout, finalize=True)

# Criar janela de resultados
def criar_janela_resultados():
    layout = [
        [sg.Multiline(size=(80, 25), key='-OUTPUT-', autoscroll=True, reroute_stdout=False, enable_events=True)],
        [sg.Button('Fechar', size=(10, 1))]
    ]
    
    return sg.Window('Resultados da Simulação', layout, finalize=True)
# Criar a janela principa
def main():

    window = criar_interface()
    resultados_window = None
    output_element = None
    
    Stop=False
    while not Stop:
        event, values = window.read()
        
        if event == sg.WINDOW_CLOSED or event == 'Sair':
            Stop=True
            
        elif event == 'Valores Padrão':
            window['-TAXA1-'].update(30)
            window['-TAXA2-'].update(12)
            window['-TAXA3-'].update(15)
            window['-TAXA4-'].update(10)
            window['-NUM_BALCOES-'].update(2)
            window['-TEMPO_BALCAO-'].update(5)
            window['-DIST_BALCAO-'].update('exponential')
            window['-NUM_MEDICOS-'].update(3)
            window['-TEMPO_CONSULTA-'].update(10)
            window['-DIST_CONSULTA-'].update('exponential')
            window['-STATUS-'].update('Valores padrão restaurados!')
            
        elif event == 'Iniciar Simulação':
            # definir parâmetros
            params = {
                'taxa_1': values['-TAXA1-'],
                'taxa_2': values['-TAXA2-'],
                'taxa_3': values['-TAXA3-'],
                'taxa_4': values['-TAXA4-'],
                'num_balcoes': int(values['-NUM_BALCOES-']),
                'tempo_medio_balcao': values['-TEMPO_BALCAO-'],
                'distribuicao_balcao': values['-DIST_BALCAO-'],
                'num_medicos': int(values['-NUM_MEDICOS-']),
                'tempo_medio_consulta': values['-TEMPO_CONSULTA-'],
                'distribuicao_consulta': values['-DIST_CONSULTA-']
            }
            
            window['-STATUS-'].update('Simulação em andamento...')
            
            # Criar janela de resultados se não existir
           
            resultados_window = criar_janela_resultados()
            output_element = resultados_window['-OUTPUT-']

            
            # Executar simulação
            simula(params, output_element)
            window['-STATUS-'].update('Simulação concluída!')
                
        
        # Verificar eventos na janela de resultados
        if resultados_window:
            result_event, result_values = resultados_window.read()
            if result_event == sg.WINDOW_CLOSED or result_event == 'Fechar':
                resultados_window.close()
                


if __name__ == "__main__":
    main()
