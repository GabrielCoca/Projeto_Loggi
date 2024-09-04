
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
```

```python
name	region	origin	vehicle_capacity	deliveries
0	cvrp-2-df-33	df-2	{'lng': -48.05498915846707, 'lat': -15.8381445...	180	[{'id': '313483a19d2f8d65cd5024c8d215cfbd', 'p...
1	cvrp-2-df-73	df-2	{'lng': -48.05498915846707, 'lat': -15.8381445...	180	[{'id': 'bf3fc630b1c29601a4caf1bdd474b85', 'po...
2	cvrp-2-df-20	df-2	{'lng': -48.05498915846707, 'lat': -15.8381445...	180	[{'id': 'b30f1145a2ba4e0b9ac0162b68d045c3', 'p...
3	cvrp-1-df-71	df-1	{'lng': -47.89366206897872, 'lat': -15.8051175...	180	[{'id': 'be3ed547394196c12c7c27c89ac74ed6', 'p...
4	cvrp-2-df-87	df-2	{'lng': -48.05498915846707, 'lat': -15.8381445...	180	[{'id': 'a6328fb4dc0654eb28a996a270b0f6e4', 'p...
```

## **4\. Exploração de Dados**

Realizarei uma exploração dos dados para identificar informações importantes e definir as etapas de manipulação necessárias:

```python
deliveries_exploded_df = deliveries_df[["deliveries"]].explode("deliveries")
deliveries_exploded_df.head(5)
```

```python
deliveries
0	{'id': '313483a19d2f8d65cd5024c8d215cfbd', 'po...
0	{'id': '320c94b17aa685c939b3f3244c3099de', 'po...
0	{'id': '3663b42f4b8decb33059febaba46d5c8', 'po...
0	{'id': 'e11ab58363c38d6abc90d5fba87b7d7', 'poi...
0	{'id': '54cb45b7bbbd4e34e7150900f92d7f4b', 'po...
```

```python
hub_origin_df = pd.json_normalize(deliveries_df["origin"])
hub_origin_df.head(5)
```

```python
	lng	lat
0	-48.054989	-15.838145
1	-48.054989	-15.838145
2	-48.054989	-15.838145
3	-47.893662	-15.805118
4	-48.054989	-15.838145
```

Observamos que as colunas *origin* e *deliveries* contêm campos multivalorados, armazenando diferentes informações em um único campo. Para corrigir isso, utilizarei o método normalize na coluna origin e, em seguida, realizarei um merge para unir os dois DataFrames. Após essa operação, a coluna origin será excluída, pois não será mais necessária no nosso DataFrame.

```python
# Normalizando a coluna 'origin' para extrair informações detalhadas
hub_origin_df = pd.json_normalize(deliveries_df["origin"])
deliveries_df = pd.merge(left=deliveries_df, right=hub_origin_df, how='inner', left_index=True, right_index=True)
deliveries_df = deliveries_df.drop("origin", axis=1)
deliveries_df = deliveries_df[["name", "region", "lng", "lat", "vehicle_capacity", "deliveries"]]
deliveries_df.rename(columns={"lng": "hub_lng", "lat": "hub_lat"}, inplace=True)
deliveries_df.head(5)
```

```python
	name	region	hub_lng	hub_lat	vehicle_capacity	deliveries
0	cvrp-2-df-33	df-2	-48.054989	-15.838145	180	[{'id': '313483a19d2f8d65cd5024c8d215cfbd', 'p...
1	cvrp-2-df-73	df-2	-48.054989	-15.838145	180	[{'id': 'bf3fc630b1c29601a4caf1bdd474b85', 'po...
2	cvrp-2-df-20	df-2	-48.054989	-15.838145	180	[{'id': 'b30f1145a2ba4e0b9ac0162b68d045c3', 'p...
3	cvrp-1-df-71	df-1	-47.893662	-15.805118	180	[{'id': 'be3ed547394196c12c7c27c89ac74ed6', 'p...
4	cvrp-2-df-87	df-2	-48.054989	-15.838145	180	[{'id': 'a6328fb4dc0654eb28a996a270b0f6e4', 'p...
```

 A coluna *deliveries* também possui um campo multivalorado, contendo informações distintas. Para resolver isso, apliquei o método explode para normalizar os dados. Em seguida, utilizei uma função lambda para extrair o campo **lng**, **lat** e **size** de cada subcampo **point** e **size**. Em sequencia, combinei os DataFrames resultantes utilizando **concat**.

 ```python
 # Expandindo a coluna 'deliveries' para normalizar as informações de entrega
deliveries_exploded_df = deliveries_df[["deliveries"]].explode("deliveries")

# Extraindo detalhes específicos de latitude e longitude das entregas
deliveries_normalized_df = pd.concat([
    pd.DataFrame(deliveries_exploded_df["deliveries"].apply(lambda record: record["size"])).rename(columns={"deliveries": "delivery_size"}),
    pd.DataFrame(deliveries_exploded_df["deliveries"].apply(lambda record: record["id"])).rename(columns={"deliveries":"delivery_id"}),
    pd.DataFrame(deliveries_exploded_df["deliveries"].apply(lambda record: record["point"]["lng"])).rename(columns={"deliveries": "delivery_lng"}),
    pd.DataFrame(deliveries_exploded_df["deliveries"].apply(lambda record: record["point"]["lat"])).rename(columns={"deliveries": "delivery_lat"}),
], axis=1)

# Integrando os dados normalizados de entrega com o DataFrame principal
deliveries_df = deliveries_df.drop("deliveries", axis=1)
deliveries_df = pd.merge(left=deliveries_df, right=deliveries_normalized_df, how='right', left_index=True, right_index=True)
deliveries_df.reset_index(inplace=True, drop=True)

# Exibindo o DataFrame final
deliveries_df.head()
```

```python

name	region	hub_lng	hub_lat	vehicle_capacity	delivery_size	delivery_id	delivery_lng	delivery_lat
0	cvrp-2-df-33	df-2	-48.054989	-15.838145	180	9	313483a19d2f8d65cd5024c8d215cfbd	-48.116189	-15.848929
1	cvrp-2-df-33	df-2	-48.054989	-15.838145	180	2	320c94b17aa685c939b3f3244c3099de	-48.118195	-15.850772
2	cvrp-2-df-33	df-2	-48.054989	-15.838145	180	1	3663b42f4b8decb33059febaba46d5c8	-48.112483	-15.847871
3	cvrp-2-df-33	df-2	-48.054989	-15.838145	180	2	e11ab58363c38d6abc90d5fba87b7d7	-48.118023	-15.846471
4	cvrp-2-df-33	df-2	-48.054989	-15.838145	180	7	54cb45b7bbbd4e34e7150900f92d7f4b	-48.114898	-15.858055
```





