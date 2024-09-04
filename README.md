
# **Storytelling para o Portfólio**: Análise Exploratória de Dados de Logística Loggi

Análise Exploratória de Dados de Logística Loggi

---

# **Índice**

<ol type="1">
  <li>Projeto;</li>
  <li>Importação de Pacotes e Bibliotecas;</li>
  <li>Carregamento dos Dados;</li>
  <li>Exploração de Dados;</li>
  <li>Análise Exploratória;</li>
  <li>Limpeza e Transformação de Dados;</li>
  <li>Visualizações;</li>
  <li>Insights Gerados;</li>
  <li>Conclusão;</li>
</ol>

---

## **1\. Projeto**

### 1.1. Contexto

Este projeto realiza uma análise detalhada dos dados logísticos da ***startup Loggi***, com foco nas operações do Distrito Federal. A análise examina a adequação da quantidade de HUBs de distribuição e a distribuição de entregas entre eles, com o objetivo de identificar oportunidades de melhoria na eficiência operacional.

A análise visa responder a duas principais questões:

*   A quantidade de HUBs de distribuição está adequada para atender a demanda?
*   A distribuição de entregas entre os HUBs é equilibrada?

### 1.2. Dado

Os dados utilizados para este projeto estão armazenados em um arquivo `JSON` contendo uma lista de instâncias de entregas. Cada instância representa um conjunto de entregas que devem ser realizadas pelos veículos de um HUB regional. Exemplo:

```json
[
  {
    "name": "cvrp-0-df-0",
    "region": "df-0",
    "origin": {"lng": -47.802664728268745, "lat": -15.657013854445248},
    "vehicle_capacity": 180,
    "deliveries": [
      {
        "id": "ed0993f8cc70d998342f38ee827176dc",
        "point": {"lng": -47.7496622016347, "lat": -15.65879313293694},
        "size": 10
      },
      {
        "id": "c7220154adc7a3def8f0b2b8a42677a9",
        "point": {"lng": -47.75887552060412, "lat": -15.651440380492554},
        "size": 10
      },
      ...
    ]
  }
]
...
```
## **2\. Importação de Pacotes e Bibliotecas**

```python
!pip install geopandas;
from getpass import getpass
from geopy.geocoders import Nominatim
from geopy.extra.rate_limiter import RateLimiter
import os
import geopy
import numpy as np
import pandas as pd
import geopandas as gpd
import json
import matplotlib.pyplot as plt
import seaborn as sns
```
## **3\. Carregamento dos Dados**

Os dados utilizados neste projeto estão disponíveis em um arquivo JSON armazenado no repositório GitHub. Abaixo, o código para clonagem do repositório e carregamento dos dados:

```python
username = "GabrielCoca"
os.environ["GITHUB_USER"] = username

!git config --global user.name "${GITHUB_USER}"

usermail = getpass()
os.environ["GITHUB_MAIL"] = usermail

!git config --global user.email "${GITHUB_MAIL}"

usertoken = getpass()
os.environ["GITHUB_TOKEN"] = usertoken

!git clone https://${GITHUB_USER}:${GITHUB_TOKEN}@github.com/GabrielCoca/Projeto_Loggi.git

%cd /content/Projeto_Loggi
```
Os dados no formato JSON e serão convertidos para um DataFrame, facilitando a manipulação durante a análise.

```python
# Carregando o dado bruto a partir do arquivo JSON
with open('base/deliveries.json', mode='r', encoding='utf8') as file:
    data = json.load(file)

# Convertendo o dado bruto em um DataFrame do pandas
deliveries_df = pd.DataFrame(data)

# Exibindo o DataFrame
deliveries_df.head()
{df_markdown1}
```

## **4\. Exploração de Dados**
