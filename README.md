# Project 3 Geospatial Analysis Project

# a.- Preprocesamiento de datos para el análisis
1
2
3
4

# b.- Análisis en profundidad de los restaurantes
8
# c.- Análsis del mejor restaurante
11

# d.-  Análisis de los precios de los restaruantes 

<hr>

<a name="schema0"></a>

# 0. INFO  del dataset

Columns description
1.url contains the url of the restaurant in the zomato website

2.address contains the address of the restaurant in Bengaluru

3.name contains the name of the restaurant

4.online_order whether online ordering is available in the restaurant or not

5.book_table table book option available or not

6.rate contains the overall rating of the restaurant out of 5

7.votes contains total number of rating for the restaurant as of the above mentioned date

8.phone contains the phone number of the restaurant

9.location contains the neighborhood in which the restaurant is located

10.rest_type restaurant type

11.dish_liked dishes people liked in the restaurant

12.cuisines food styles, separated by comma

13.approx_cost(for two people) contains the approximate cost for meal for two people

14.reviews_list list of tuples containing reviews for the restaurant, each tuple

15.menu_item contains list of menus available in the restaurant

16.listed_in(type) type of meal

17.listed_in(city) contains the neighborhood in which the restaurant is listed20. [Enlaces ](#schema20)

<hr>

<a name="schema1"></a>

# 1. Importar librerías y cargar los datos
~~~python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
df=pd.read_csv('./data/zomato.csv')
~~~
![img](./images/001.png)

<hr>

<a name="schema2"></a>

# 2. Limpiar el data set
~~~pyhton
df.shape
df.dtypes
len(df['name'].unique())
df.isna().sum()
~~~
Tenemos `51717`datos en `17`
Todos los datos on `objects`
Los nombres diferences son `8792`
Los nulos 

![img](./images/002.png)

<hr>

<a name="schema3"></a>

# 3. Obtener las datos con NAN
~~~python
feature_na=[feature for feature in df.columns if df[feature].isnull().sum()>0]
~~~
![img](./images/003.png)
~~~python
for feature in feature_na:
    print('{} has {} % missing values'.format(feature,np.round(df[feature].isnull().sum()/len(df)*100,4)))
~~~
![img](./images/004.png)

Comprobamos que la columna con mas valores nulos es la `rate`

<hr>

<a name="schema4"></a>

# 4. Vamos a modificar la columna rate, quitandole los nulos y variando su formatos
~~~python
df['rate'].unique()
~~~
![img](./images/005.png)
~~~python
df.dropna(axis='index',subset=['rate'],inplace=True)
~~~
Al eliminar los nulos de rate son quedamos con estos datos:
![img](./images/006.png)

Para variar el formato de la columna creamos una función:
~~~python
def split(x):
    return x.split('/')[0]
df['rate']=df['rate'].apply(split)    
~~~
![img](./images/007.png)

Después remplazamos los valores que contengan `NEW` y `-` por ceros y convertimos todos los valores a`float `
~~~python
df.replace('NEW',0,inplace=True)
df.replace('-',0,inplace=True)
df['rate']=df['rate'].astype(float)
~~~
<hr>

<a name="schema5"></a>

# 5 . Calcular la calificación promedio de cada restaurante
Agrupamos por nombre y hacemos la media del `rate` y mostramos los 20 mejores

~~~python
df.groupby('name')['rate'].mean().nlargest(20).plot.bar()
plt.savefig("./images/name.png")
~~~
![img](./images/name.png)
~~~python
df_rate=df.groupby('name')['rate'].mean().to_frame()
df_rate=df_rate.reset_index()
df_rate.columns=['restaurant','rating']
df_rate.head(20)
~~~
<hr>

<a name="schema6"></a>

# 6. Dibujar la distribución del rating
~~~python
sns.set_style(style='whitegrid')
sns.distplot(df_rate['rating'])
~~~
![img](./images/rating.png)

Casi más del 50 por ciento de los restaurantes tienen una calificación de entre 3 y 4. Los restaurantes con una calificación de más de 4.5 son muy raros.

<hr>

<a name="schema7"></a>

# 7 .¿Cuáles son las principales cadenas de restaurantes en Bangalore?
~~~python
plt.figure(figsize=(10,7))
chains=df['name'].value_counts()[0:20]
sns.barplot(x=chains,y=chains.index,palette='deep')
plt.title("Most famous restaurants chains in Bangaluru")
plt.xlabel("Number of outlets") 
~~~
![img](./images/most.png)


<hr>

<a name="schema8"></a>


# 8. ¿Cuántos de los restaurantes no aceptan pedidos online?
~~~python
x=df['online_order'].value_counts()
labels=['accepted','not accepted']
plt.pie(x,explode=[0.0,0.1],autopct='%1.1f%%')
~~~
![img](./images/online.png)


<hr>

<a name="schema9"></a>

# 9. ¿Cuál es la relación entre restaurantes que ofrecen y no ofrecen reserva de mesa?
~~~python
x=df['book_table'].value_counts()
labels=['not book','book']
plt.pie(x,explode=[0.0,0.1],autopct='%1.1f%%')
~~~
![img](./images/table.png)

<hr>

<a name="schema10"></a>

# 10. Limpiamos los datos

~~~python
df['rest_type'].isna().sum()
df['rest_type'].dropna(inplace=True)
plt.figure(figsize=(20,12))
df['rest_type'].value_counts().nlargest(20).plot.bar(color='red')
plt.gcf().autofmt_xdate()
plt.savefig("./images/rest_type.png")
~~~
![img](./images/rest_type.png)


<hr>

<a name="schema11"></a>

# 11. Los restaurantes mas votados y total de restaurantes en diferentes ubicaciones de Bangalore
~~~python
df.groupby('name')['votes'].max().nlargest(10).plot.bar()
plt.savefig("./images/votes.png")
~~~

![img](./images/votes.png)

~~~python
df.groupby('location')['name'].unique()
~~~
![img](./images/009.png)


Creamos un diccionario con las localizaciones y la cantidad de resturantes. 
~~~python
restaurant=[]
location=[]
for key,location_df in df.groupby('location'):
    location.append(key)
    restaurant.append(len(location_df['name'].unique()))
df_total=pd.DataFrame(zip(location,restaurant))
df_total.columns=['location','restaurant']
df_total.set_index('location',inplace=True)
df_total.sort_values(by='restaurant').tail(10)
~~~
![img](./images/010.png)


~~~python
df_total.sort_values(by='restaurant').tail(10).plot.bar()
plt.savefig("./images/sort_rest.png")
~~~
![img](./images/sort_rest.png)


<hr>

<a name="schema12"></a>

# 12. Número total de variedad de restaurantes, es decir, norte de la India, sur de la India
~~~python
cuisines=df['cuisines'].value_counts()[:10]
sns.barplot(cuisines,cuisines.index)
plt.xlabel('Count')
plt.title("Most popular cuisines of Bangalore")
~~~
![img](./images/cuisines.png)

Comprobamos que la mayor variedad de restaurantes son `North India` y `North India Chinese`

<hr>

<a name="schema13"></a>

# 13. Analizar costo aproximado para 2 personas

Comprobamos si hay nulos, y si los hay los eliminamos.

~~~python
df["approx_cost(for two people)"].isna().sum()
~~~
![img](./images/011.png)
~~~python
df.dropna(axis='index',subset=['approx_cost(for two people)'],inplace=True)
~~~
Como los dastos son `object` los converitmos a `int`, primero les quitamos las comas y después los pasamos a `int`. Los podemos pasar a `int` porque hemos comprobado que no hay ningún dato tenga decimales.

~~~python


df['approx_cost(for two people)'] = df['approx_cost(for two people)'].apply(lambda x: x.replace(',',''))
df['approx_cost(for two people)']=df['approx_cost(for two people)'].astype(int)
~~~
<hr>

<a name="schema14"></a>

# 14.  Gasto vs rating

~~~python
plt.figure(figsize=(10,7))
sns.scatterplot(x="rate",y='approx_cost(for two people)',hue='online_order',data=df)
plt.show()

~~~
![img](./images/costvsrating.png)






