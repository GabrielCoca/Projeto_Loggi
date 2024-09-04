
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

## **5\. Análise Exploratória**

Agora, com o Dataframe estruturado, partiremos para uma analise exploratória.

```python
# Dimensões do DataFrame
deliveries_df.shape
```

```python
(636149, 9)
```

```python
# Tipo de dado do DataFrame
deliveries_df.info()
```

```python
<class 'pandas.core.frame.DataFrame'>
RangeIndex: 636149 entries, 0 to 636148
Data columns (total 9 columns):
 #   Column            Non-Null Count   Dtype  
---  ------            --------------   -----  
 0   name              636149 non-null  object 
 1   region            636149 non-null  object 
 2   hub_lng           636149 non-null  float64
 3   hub_lat           636149 non-null  float64
 4   vehicle_capacity  636149 non-null  int64  
 5   delivery_size     636149 non-null  int64  
 6   delivery_id       636149 non-null  object 
 7   delivery_lng      636149 non-null  float64
 8   delivery_lat      636149 non-null  float64
dtypes: float64(4), int64(2), object(3)
memory usage: 43.7+ MB
```

```python
# Identificando valores nulos
deliveries_df.isna().sum()
```

```python
name                  0
region                0
hub_lng               0
hub_lat               0
vehicle_capacity      0
delivery_size         0
delivery_id           0
delivery_lng          0
delivery_lat          0
dtype: int64
```

```python
# Contando valores distindo nas colunas
deliveries_df.nunique()
```

```python

0
name	199
region	3
hub_lng	3
hub_lat	3
vehicle_capacity	1
delivery_size	10
delivery_id	291566
delivery_lng	291566
delivery_lat	291566
dtype: int64
```

```python
# Separando os dados numéricos dos dados categóricos
var_numericos = []
var_categoricas = []

for coluna in deliveries_df.columns:
  if deliveries_df[coluna].dtype != "object":
    var_numericos.append(coluna)
  else:
    var_categoricas.append(coluna)

print(f'As colunas numéricas são: {var_numericos}')
print(f'As colunas categóricas são: {var_categoricas}')
```

```python
As colunas numéricas são: ['hub_lng', 'hub_lat', 'vehicle_capacity', 'delivery_size', 'delivery_lng', 'delivery_lat']
As colunas categóricas são: ['name', 'region', 'delivery_id']

```python
# Resumo statístico
deliveries_df[var_numericos].describe().round(2)
```

```python
	hub_lng	hub_lat	vehicle_capacity	delivery_size	delivery_lng	delivery_lat
count	636149.00	636149.00	636149.0	636149.00	636149.00	636149.00
mean	-47.95	-15.80	180.0	5.51	-47.95	-15.81
std	0.09	0.05	0.0	2.87	0.11	0.08
min	-48.05	-15.84	180.0	1.00	-48.28	-16.05
25%	-48.05	-15.84	180.0	3.00	-48.04	-15.84
50%	-47.89	-15.81	180.0	6.00	-47.93	-15.81
75%	-47.89	-15.81	180.0	8.00	-47.88	-15.77
max	-47.80	-15.66	180.0	10.00	-47.31	-15.50
```

- **Nota:**

Na coluna vehicle_capacity, todos os valores são iguais a 180, indicando que a capacidade dos veículos é uniforme. Já a coluna delivery_size apresenta uma variação moderada, com média de 5,51 e mediana de 6.

```python
# Resumo estatistico
deliveries_df[var_categoricas].describe().round(2)
```

```python
	name	region	delivery_id
count	636149	636149	636149
unique	199	3	291566
top	cvrp-1-df-87	df-1	61b87669243974d021c2b76fc5272045
freq	5636	304708	12
```

- **Nota:**

O Distrito Federal contém três HUBs, sendo o HUB df-1 o que possui o maior número de entregas.

```python
# Análise de proporção de entregas por região
data = pd.DataFrame(deliveries_df[['region', 'vehicle_capacity']].value_counts(normalize=True)).reset_index()
data.rename(columns={0: "proportion"}, inplace=True)
data["proportion"] = (data["proportion"] * 100).round(2)
data.head()
```

```python
	region	vehicle_capacity	proportion
0	df-1	180	47.90
1	df-2	180	41.08
2	df-0	180	11.02
```

- **Nota:**

Os HUBs um e dois concentram mais de 80% das entregas no Distrito Federal. Esse dado sugere que aumentar a participação do HUB zero pode melhorar a distribuição de entregas entre os três HUBs.

A **geocodificação** é o processo de converter uma descrição textual de um local (como um endereço ou nome de um lugar) em suas coordenadas geográficas correspondentes (latitude e longitude). Já a **geocodificação reversa** faz o inverso: transforma uma coordenada geográfica em sua descrição textual correspondente.

```python
# Inicializando o geolocalizador
geolocator = Nominatim(user_agent="Projeto_Loggi_Geocoder")
geocoder = RateLimiter(geolocator.reverse, min_delay_seconds=1)
```

Normalização e tratamento de campos multivalorados:

```python
# Criando um DataFrame apenas com as coordenadas dos hubs
hub_df = deliveries_df[["region", "hub_lng", "hub_lat"]]
hub_df = hub_df.drop_duplicates().sort_values(by="region").reset_index(drop=True)
hub_df["coordinates"] = hub_df["hub_lat"].astype(str)  + ", " + hub_df["hub_lng"].astype(str)

# Aplicando o geocoder para obter informações geográficas detalhadas
hub_df["geodata"] = hub_df["coordinates"].apply(geocoder)

# Normalizando e organizando os dados geográficos dos hubs
hub_geodata_df = pd.json_normalize(hub_df["geodata"].apply(lambda data: data.raw))

# Selecionamos apenas as colunas relevantes (cidade, subúrbio, e cidade principal)
hub_geodata_df = hub_geodata_df[["address.town", "address.suburb", "address.city"]]
hub_geodata_df.rename(columns={"address.town": "hub_town", "address.suburb": "hub_suburb", "address.city": "hub_city"}, inplace=True)
hub_geodata_df["hub_city"] = np.where(hub_geodata_df["hub_city"].notna(), hub_geodata_df["hub_city"], hub_geodata_df["hub_town"])
hub_geodata_df["hub_suburb"] = np.where(hub_geodata_df["hub_suburb"].notna(), hub_geodata_df["hub_suburb"], hub_geodata_df["hub_city"])
hub_geodata_df = hub_geodata_df.drop("hub_town", axis=1)

# Visualização após consolidação principal
hub_geodata_df.head()
```

```python
	hub_suburb	hub_city
0	Sobradinho	Sobradinho
1	Asa Sul	Brasília
2	Taguatinga	Taguatinga
```

```python
# Mesclando informações geográficas detalhadas dos hubs com o DataFrame principal
hub_df = pd.merge(left=hub_df, right=hub_geodata_df, left_index=True, right_index=True)
hub_df = hub_df[["region", "hub_suburb", "hub_city"]]
deliveries_df = pd.merge(left=deliveries_df, right=hub_df, how="inner", on="region")
deliveries_df = deliveries_df[["name", "region", "hub_lng", "hub_lat", "hub_city", "hub_suburb", "vehicle_capacity", "delivery_size", "delivery_id", "delivery_lng", "delivery_lat"]]
deliveries_df.head()
```

```python

name	region	hub_lng	hub_lat	hub_city	hub_suburb	vehicle_capacity	delivery_size	delivery_id	delivery_lng	delivery_lat
0	cvrp-2-df-33	df-2	-48.054989	-15.838145	Taguatinga	Taguatinga	180	9	313483a19d2f8d65cd5024c8d215cfbd	-48.116189	-15.848929
1	cvrp-2-df-33	df-2	-48.054989	-15.838145	Taguatinga	Taguatinga	180	2	320c94b17aa685c939b3f3244c3099de	-48.118195	-15.850772
2	cvrp-2-df-33	df-2	-48.054989	-15.838145	Taguatinga	Taguatinga	180	1	3663b42f4b8decb33059febaba46d5c8	-48.112483	-15.847871
3	cvrp-2-df-33	df-2	-48.054989	-15.838145	Taguatinga	Taguatinga	180	2	e11ab58363c38d6abc90d5fba87b7d7	-48.118023	-15.846471
4	cvrp-2-df-33	df-2	-48.054989	-15.838145	Taguatinga	Taguatinga	180	7	54cb45b7bbbd4e34e7150900f92d7f4b	-48.114898	-15.858055
```

```python
# Carregando dados de geolocalização adicionais de entregas de um arquivo CSV
deliveries_geodata_df = pd.read_csv("base/deliveries-geodata.csv")

# Mesclando os dados de geolocalização das entregas com o DataFrame principal
deliveries_df = pd.merge(left=deliveries_df, right=deliveries_geodata_df[["delivery_city", "delivery_suburb"]], how="inner", left_index=True, right_index=True)
deliveries_df.head()
```

```python
name	region	hub_lng	hub_lat	hub_city	hub_suburb	vehicle_capacity	delivery_size	delivery_id	delivery_lng	delivery_lat	delivery_city	delivery_suburb
0	cvrp-2-df-33	df-2	-48.054989	-15.838145	Taguatinga	Taguatinga	180	9	313483a19d2f8d65cd5024c8d215cfbd	-48.116189	-15.848929	Ceilândia	P Sul
1	cvrp-2-df-33	df-2	-48.054989	-15.838145	Taguatinga	Taguatinga	180	2	320c94b17aa685c939b3f3244c3099de	-48.118195	-15.850772	Ceilândia	P Sul
2	cvrp-2-df-33	df-2	-48.054989	-15.838145	Taguatinga	Taguatinga	180	1	3663b42f4b8decb33059febaba46d5c8	-48.112483	-15.847871	Ceilândia	P Sul
3	cvrp-2-df-33	df-2	-48.054989	-15.838145	Taguatinga	Taguatinga	180	2	e11ab58363c38d6abc90d5fba87b7d7	-48.118023	-15.846471	Ceilândia	P Sul
4	cvrp-2-df-33	df-2	-48.054989	-15.838145	Taguatinga	Taguatinga	180	7	54cb45b7bbbd4e34e7150900f92d7f4b	-48.114898	-15.858055	Sol Nascente/Pôr do Sol	Sol Nascente/Pôr do Sol
```

```python
# Download dos dados geográficos do Distrito Federal
!wget -q "https://geoftp.ibge.gov.br/cartas_e_mapas/bases_cartograficas_continuas/bc100/go_df/versao2016/shapefile/bc100_go_df_shp.zip" -O distrito-federal.zip
!unzip -q distrito-federal.zip -d ./maps
!cp ./maps/LIM_Unidade_Federacao_A.shp ./distrito-federal.shp
!cp ./maps/LIM_Unidade_Federacao_A.shx ./distrito-federal.shx

# Leitura e visualização do mapa do Distrito Federal
mapa = gpd.read_file("distrito-federal.shp")
mapa = mapa.loc[[0]]
mapa.head()
```

```python
geometry
0	POLYGON Z ((-47.31048 -16.03602 0.00000, -47.3...
```

Este código prepara um conjunto de dados de HUBs de distribuição, eliminando duplicações e convertendo-o em um formato geoespacial (GeoDataFrame). Isso permite realizar análises geoespaciais, como visualização de mapas ou cálculos de distâncias entre os pontos de entrega.

```python
# Criação de GeoDataFrames para os hubs
hub_df = deliveries_df[["region", "hub_lng", "hub_lat"]].drop_duplicates().reset_index(drop=True)
geo_hub_df = gpd.GeoDataFrame(hub_df, geometry=gpd.points_from_xy(hub_df["hub_lng"], hub_df["hub_lat"]))
geo_hub_df.head()
```

```python
	region	hub_lng	hub_lat	geometry
0	df-2	-48.054989	-15.838145	POINT (-48.05499 -15.83814)
1	df-1	-47.893662	-15.805118	POINT (-47.89366 -15.80512)
2	df-0	-47.802665	-15.657014	POINT (-47.80266 -15.65701)
```

```python	
# Criação de GeoDataFrames para as entregas
geo_deliveries_df = gpd.GeoDataFrame(deliveries_df, geometry=gpd.points_from_xy(deliveries_df["delivery_lng"], deliveries_df["delivery_lat"]))
geo_deliveries_df.head()
```

```python
name	region	hub_lng	hub_lat	hub_city	hub_suburb	vehicle_capacity	delivery_size	delivery_id	delivery_lng	delivery_lat	delivery_city	delivery_suburb	geometry
0	cvrp-2-df-33	df-2	-48.054989	-15.838145	Taguatinga	Taguatinga	180	9	313483a19d2f8d65cd5024c8d215cfbd	-48.116189	-15.848929	Ceilândia	P Sul	POINT (-48.11619 -15.84893)
1	cvrp-2-df-33	df-2	-48.054989	-15.838145	Taguatinga	Taguatinga	180	2	320c94b17aa685c939b3f3244c3099de	-48.118195	-15.850772	Ceilândia	P Sul	POINT (-48.11819 -15.85077)
2	cvrp-2-df-33	df-2	-48.054989	-15.838145	Taguatinga	Taguatinga	180	1	3663b42f4b8decb33059febaba46d5c8	-48.112483	-15.847871	Ceilândia	P Sul	POINT (-48.11248 -15.84787)
3	cvrp-2-df-33	df-2	-48.054989	-15.838145	Taguatinga	Taguatinga	180	2	e11ab58363c38d6abc90d5fba87b7d7	-48.118023	-15.846471	Ceilândia	P Sul	POINT (-48.11802 -15.84647)
4	cvrp-2-df-33	df-2	-48.054989	-15.838145	Taguatinga	Taguatinga	180	7	54cb45b7bbbd4e34e7150900f92d7f4b	-48.114898	-15.858055	Sol Nascente/Pôr do Sol	Sol Nascente/Pôr do Sol	POINT (-48.11490 -15.85805)
```

## **7\. Visualizações**

Visualizações gráficas para ilustrar insights importantes:

```python
# criando o plot vazio
fig, ax = plt.subplots(figsize = (50/2.54, 50/2.54))

# plot mapa do distrito federal
mapa.plot(ax=ax, alpha=0.4, color="lightgrey")

# plot das entregas
geo_deliveries_df.query("region == 'df-0'").plot(ax=ax, markersize=1, color="seagreen", label="df-0")
geo_deliveries_df.query("region == 'df-1'").plot(ax=ax, markersize=1, color="blue", label="df-1")
geo_deliveries_df.query("region == 'df-2'").plot(ax=ax, markersize=1, color="red", label="df-2")

# plot dos hubs
geo_hub_df.plot(ax=ax, markersize=30, marker="x", color="black", label="hub")

# plot da legenda
plt.title("Entregas no Distrito Federal por Região", fontdict={"fontsize": 16})
lgnd = plt.legend(prop={"size": 15})
for handle in lgnd.legendHandles:
    handle.set_sizes([50])

# Salvando a imagem em PNG no diretório especificado
plt.savefig("/content/Projeto_Loggi/base/entregas_distrito_federal.png", format="png", dpi=300, bbox_inches="tight")

plt.show()
```

![entregas_distrito_federal.png](content/Projeto_Loggi/base)

```python
# Plotando gráfico de barras para mostrar a proporção de entregas por região
plt.figure(figsize=(10, 6))
with sns.axes_style('whitegrid'):
  grafico = sns.barplot(data=data, x="region", y="proportion", ci=None, palette="pastel")
  grafico.set(title='Proporção de entregas por região', xlabel='Região', ylabel='Proporção')

# Salvando a imagem em PNG no diretório especificado
plt.savefig("/content/Projeto_Loggi/base/proporcao_entregas_por_regiao.png.png", format="png", dpi=300, bbox_inches="tight")

plt.show()
```

![proporcao_entregas_por_regiao.png](content/Projeto_Loggi/base)

```python
plt.figure(figsize=(10, 6))
sns.countplot(data=deliveries_df, x='region', palette='pastel', order=['df-1', 'df-2', 'df-0'])
plt.title('Distribuição de Entregas por Região')
plt.xlabel('Região')
plt.ylabel('Contagem de Entregas')

# Salvando a imagem em PNG no diretório especificado
plt.savefig("/content/Projeto_Loggi/base/contagem_entregas_por_regiao..png", format="png", dpi=300, bbox_inches="tight")
plt.show()
```

![contagem_entregas_por_regiao..png](content/Projeto_Loggi/base)

## **8\. Insights Gerados**


*   As entregas estão devidamente alocadas aos seus respectivos HUBs, mas os HUBs das regiões DF-0 e DF-2 realizam entregas em locais mais distantes do centro e entre si, o que pode aumentar o tempo e o custo de entrega. Observamos também que há Regiões Administrativas com baixa cobertura de atuação dos HUBs. O HUB DF-1, por outro lado, opera em uma área mais concentrada, o que sugere que os HUBs DF-0 e DF-2 enfrentam custos operacionais potencialmente mais elevados, especialmente em termos de consumo de combustível.

*   A distribuição das entregas está fortemente concentrada nos HUBs das regiões DF-1 e DF-2, enquanto o HUB da região DF-0 tem uma participação significativamente menor. No entanto, como a capacidade dos veículos é a mesma para todos os HUBs, seria possível realocar veículos para as regiões com maior demanda de entregas. Observamos que o HUB DF-0 possui uma baixa taxa de atuação, sugerindo que há oportunidades para expandir sua presença nas Regiões Administrativas, especialmente considerando a densidade urbana dessas áreas.

### 8.1 Próximas Etapas:


*   Investigar o impacto da redistribuição de entregas entre os HUBs na eficiência operacional.

*   Utilizar modelos de otimização para simular diferentes cenários de alocação de recursos.

*   Explorar estratégias de aumento da capacidade de entrega nos HUBs menos utilizados.

## **9\. Conclusão**

Este estudo utiliza análise exploratória de dados e *storytelling* para otimizar operações logísticas, analisando a redistribuição de entregas entre HUBs e seu impacto na eficiência e nos custos operacionais. Observa-se que os HUBs das regiões DF-0 e DF-2 enfrentam maiores custos devido à distância entre os pontos de entrega, enquanto o HUB DF-1 opera de forma mais centralizada, com menores custos. A concentração de entregas nos HUBs DF-1 e DF-2 indica a necessidade de uma realocação estratégica de recursos e a expansão do HUB DF-0 para melhorar a cobertura e eficiência em áreas menos atendidas.
