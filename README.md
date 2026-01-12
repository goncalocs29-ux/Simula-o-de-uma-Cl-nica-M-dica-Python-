# Projeto simulação hospitalar 

- Na inicialização do programa projeto_sim_hospital.py, surge um painel de configuração da simulação hospitalar, no qual se podem editar vários parâmetros que influenciam os resultados obtidos.

<img width="541" height="930" alt="Captura de ecrã 2026-01-01 133824" src="https://github.com/user-attachments/assets/9dfce395-1734-4bd2-8ecc-5a677344a97d" />

- É possível variar as taxas de chegada de forma a aproximar a simulação de diferentes situações reais, em diversos contextos, sendo possível ajustar as taxas de chegada de pacientes das ondas definidas entre os valores de 10 e 30 pacientes por hora.

<img width="537" height="422" alt="Captura de ecrã 2026-01-01 133842 (1)" src="https://github.com/user-attachments/assets/077edaa7-eb30-4ae0-91c5-f742398668bf" />

- É também permitido alterar o número de balcões (1–5 balcões), o tempo médio de atendimento (1–15 minutos) e o tipo de distribuição estatística utilizada para definir o tempo de atendimento (exponencial, normal ou uniforme).

<img width="530" height="183" alt="Captura de ecrã 2026-01-01 133849" src="https://github.com/user-attachments/assets/4b907ea3-caa6-49ad-b77a-6a5d0511b739" />

- Da mesma forma, é possível editar parâmetros idênticos relativamente às consultas, sendo estes o número de médicos (1–5 médicos), o tempo médio das consultas (5–30 minutos) e o tipo de distribuição estatística utilizada para definir o tempo de consulta (exponencial, normal ou uniforme).

<img width="525" height="182" alt="Captura de ecrã 2026-01-01 133858" src="https://github.com/user-attachments/assets/0ade3495-e254-41c2-9f65-5ae8fdc8e6ba" />

- Na parte inferior da interface encontra-se o botão para iniciar a simulação com os parâmetros definidos, um botão para restaurar os valores padrão recomendados e um botão para fechar a janela e o programa. É também apresentado o estado do programa em letras amarelas por baixo dos botões.

<img width="418" height="79" alt="Captura de ecrã 2026-01-01 134330" src="https://github.com/user-attachments/assets/0882fecf-787e-4b3f-9196-debcffd355c6" />

<img width="544" height="34" alt="Captura de ecrã 2026-01-01 161322" src="https://github.com/user-attachments/assets/879b0d6c-cebd-40bb-bf5a-a6c73381f33f" />

<img width="433" height="28" alt="image-4" src="https://github.com/user-attachments/assets/bac14e60-1d10-4bf4-9755-b1d8977023b3" />

- Após o início da simulação, os resultados são apresentados numa nova janela que contém diversos dados estatísticos.

- No início da página são descritos os valores de cada parâmetro necessário para o funcionamento da simulação.

<img width="607" height="263" alt="Captura de ecrã 2026-01-01 143223-1" src="https://github.com/user-attachments/assets/2a48dcd1-a172-4361-b899-b64fe810e46a" />

- De seguida, na mesma página, são documentados os eventos por ordem temporal. A cada paciente são atribuídos um nome, idade, sexo e tipo de urgência. É registado o tempo de chegada do paciente, o tempo de saída do balcão, o tempo de início da consulta e o tempo de saída da consulta. São ainda apresentados os nomes dos pacientes nas filas do balcão e das consultas, organizados em função do tempo de chegada e do tipo de urgência, bem como o tamanho respetivo de cada fila.

<img width="602" height="477" alt="image-6" src="https://github.com/user-attachments/assets/276962ef-6296-45a3-96cb-a04cb410f1c5" />

<img width="604" height="476" alt="Captura de ecrã 2026-01-01 143214" src="https://github.com/user-attachments/assets/1aa9e0ad-5826-4f33-b604-63ab7c53c97c" />

- Posteriormente à impressão dos eventos, são disponibilizados diferentes dados estatísticos.

<img width="563" height="333" alt="Captura de ecrã 2026-01-01 184108" src="https://github.com/user-attachments/assets/c6376714-95d1-477f-9329-5080ba839640" />

<img width="561" height="198" alt="Captura de ecrã 2026-01-01 184124" src="https://github.com/user-attachments/assets/77cd5d17-39f6-4039-9015-c942c7743e3a" />

- São também apresentados alguns gráficos contendo outros dados estatísticos:

- Gráficos que demonstram a evolução do tamanho da fila ao longo do tempo da simulação, tanto no balcão como na fila de espera das consultas e outro gráfico correspondente à evolução da percentagem de ocupação dos médicos ao longo do tempo.

<img width="2232" height="731" alt="image-2" src="https://github.com/user-attachments/assets/861d2706-594c-4e40-90f7-4687e4a36b4e" />

- Gráfico de comparação das proporções entre os tamanhos das filas do balcão, sobrepondo os dados num único gráfico com a mesma escala.

<img width="1726" height="817" alt="image-3" src="https://github.com/user-attachments/assets/e52e5a67-0d06-4f10-b363-2f807399613b" />

- Gráficos que relacionam a evolução da taxa de chegada da simulação com o tamanho das filas do balcão e das consultas. É representada uma linha tracejada correspondente a uma regressão linear da relação entre as variáveis, de forma a tornar mais evidente a influência da taxa de chegada no tamanho das filas.

<img width="1784" height="881" alt="image-1" src="https://github.com/user-attachments/assets/c92c3645-24b0-4a84-9984-cb4bf8debde5" />

- Gráfico que apresenta a evolução da taxa de chegada ao longo do tempo.

<img width="1494" height="817" alt="image" src="https://github.com/user-attachments/assets/11e85dd4-0f21-4215-8819-d2dd04770194" />

- Em cada gráfico com a variável tempo no eixo horizontal x, é representada uma linha vertical tracejada e um posterior preenchimento a vermelho, indicando o fim das 8 horas de simulação de geração de chegadas estabelecidas e a consequente anulação da taxa de chegada. Esta representação permite identificar a capacidade de escoamento das filas e perceber quais os ajustes necessários para minimizar os tempos de espera e evitar a sobrecarga do sistema hospitalar.

- Resumidamente, o projeto foi idealizado para simular o funcionamento de um hospital de forma relativamente realista, proporcionando flexibilidade para representar diferentes situações hospitalares, com características distintas, e apresentando estatísticas úteis para a avaliação da influência de cada parâmetro no fluxo de pacientes.


## Exemplo de aplicação da simulação numa análise

- Suponhamos que pretendemos descobrir qual o número mínimo de médicos necessário para escoar as filas de forma eficiente, quando o tempo médio de consulta assume um determinado valor, evitando ao máximo o prolongamento do funcionamento após as 8 horas de simulação, mas mantendo simultaneamente uma elevada percentagem de ocupação e um tamanho médio da fila de espera o mais reduzido possível.

- Começamos por manter os valores padrão na maioria dos parâmetros, alterando apenas o tempo médio de consulta para o valor desejado, e fazemos variar o número de médicos.

- Iniciamos a simulação e observamos o gráfico da evolução do tamanho da fila de consultas e o gráfico da evolução da ocupação dos médicos.

- Por fim, analisamos os dados estatísticos finais no separador de resultados.

- Voltamos a alterar os parâmetros pretendidos e repetimos o processo.

### Exemplo de resultados para um tempo médio de consulta de 15 minutos:

- **2 médicos:**

 <img width="1609" height="926" alt="Captura de ecrã 2026-01-01 174213-1" src="https://github.com/user-attachments/assets/fe4bd38c-9b34-46ab-91c7-08cc4b2bc14a" />

- **3 médicos:**

 <img width="1626" height="892" alt="Captura de ecrã 2026-01-01 174659-1" src="https://github.com/user-attachments/assets/0f14f210-05ab-4675-9abc-830b978e441f" />

- **4 médicos:**

<img width="1636" height="928" alt="Captura de ecrã 2026-01-01 175702-1" src="https://github.com/user-attachments/assets/4ef17ea9-44ca-401a-a30e-dd85da4a2c3e" />

- **5 médicos:**

<img width="1641" height="922" alt="Captura de ecrã 2026-01-01 174424-1" src="https://github.com/user-attachments/assets/0ce37a71-893c-4dd8-bc82-95997e0e8458" />

- É possível observar, de forma geral, uma tendência progressiva para a diminuição do tempo de ocupação, do tamanho das filas e do tempo necessário após o fim da geração de chegadas. Verifica-se também um aumento acentuado do tempo após o fim da geração de chegadas quando o número de médicos é apenas 2, concluindo-se que o número mínimo de médicos aconselhável para evitar a sobrecarga do sistema hospitalar é 3.


- É importante realçar que a determinação dos tempos de chegada, de atendimento ao balcão e da duração das consultas é efetuada com base em distribuições estatísticas. Deste modo existe alguma variabilidade nos resultados entre simulações distintas com parâmetros idênticos, o que deve ser tido em conta na análise e avaliação dos resultados.
