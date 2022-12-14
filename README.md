# Projeto TEMTEM
- Esse projeto tem como objetivo treinar minhas habilidades em python e ciência de dados
- Vamos usar a API do temtem https://temtem-api.mael.tech
- Alguns dos objetivos
    - Organizar numa tabela informações principais dos temtem
    - Organizar em outra tabela ou arquivo informações mais detalhadas de TemTem
    - Descobrir os valores médios dos atributos de cada tipo de temtem
    - Descobrir os temtem que tem os melhores valores de cada atributo (por exemplo, o temtem que tem mais HP)
    - Mais detalhes para o futuro
   
# Sobre TEMTEM
- Temtem é um jogo de captura e treinamento de monstros conhecidos como temtems, treinamos esses monstros para participar de batalhas e ser o melhor domador de temtems
- Atualmente tem 164 Temtems
- Cada temtem tem propriedades que são:
    - Tipo do temtem
    - Atributos de batalha
    - Movimentos de batalha
    - Detalhes como altura e peso
    - Um campo de trivia
    - Evoluções sendo a primeira evolução stage 1
    - Localização do temtem
    - Um retrato do temtem
    - A chance de domar um temtem
    - Tempo que demora para nascer um ovo do temtem
- Queremos descobrir mais informações dos temtem para tornar um domador melhor
- Um temtem pode ter somente 1 trait de dois possíveis traits

# Escolhas do projeto
- Foi escolhido que não é importante saber as técnicas de combate que os temtem aprendem, apenas informações sobre seus atributos
- Na parte de evolução só é importante saber se o temtem evolui ou não, não precisamos de detalhes da sua evolução
- Detalhes da evolução pode ser guardade em uma nova tabela para futuras avaliações se necessário


```python
import requests
import pandas as pd
import numpy as np
```

#### requests.get()
- A função requests.get() retorna um objeto de resposta, podemos usar o atributo resposta.status_code para receber o código de resposta de nossa resposta

### Tipos de resposta de api
- 200: Aconteceu tudo certo e o resultado foi retornado (se tiver resultado pra ser retornado)
- 301: O servidor está redirecionado você para um novo endpoint. Isso pode acontecer com empresas que trocam o nome do domínio ou o nome de um endpoint foi trocado
- 400: Esse servidor acha que você fez um bad request. Isso pode acontecer quando você não manda os dados certos entre outrasc coisas
- 401: Esse servidor acha que você não está autenticado. Muitas API's precisam de um login de credenciais, então isso acontece quando você não tem as credenciais certas para acessar a API
- 403: O recurso que você está tentando acessar é poibido, e você não ter as permissões certas para ver
- 404: O recurso que você está tentando acessar não existe no servidor
- 503: O servidor não está pronto para esse tipo de request

### Informações que queremos guardar do JSON do TEMTEM
- number
- name
- types
- stats
- traits
- details
- techniques
- evolution
- genderRatio
- catchRate
- hatchMins
- tvYields (esse campo é o quanto de tv o seu temtem ganha ao derrotar esse temtem em uma batalha)


```python
# Criando um dicionário para organizar as informações necessárias
def cria_dict(resposta_json):
    dict_info_temtems = {
        'number': resposta_json['number'],
        'name': resposta_json['name'],
        'hp': resposta_json['stats']['hp'],
        'sta': resposta_json['stats']['sta'],
        'spd': resposta_json['stats']['spd'],
        'atk': resposta_json['stats']['atk'],
        'def': resposta_json['stats']['def'],
        'spatk': resposta_json['stats']['spatk'],
        'spdef': resposta_json['stats']['spdef'],
        'total': resposta_json['stats']['total'],
        'height_cm': resposta_json['details']['height']['cm'],
        'weight_kg': resposta_json['details']['weight']['kg'],
        'trait 1': resposta_json['traits'][0],
        'trait 2': resposta_json['traits'][1],
        'male': resposta_json['genderRatio']['male'],
        'female': resposta_json['genderRatio']['female'],
        'catchRate': resposta_json['catchRate'],
        'hatchMins': resposta_json['hatchMins'],
        'tvHP': resposta_json['tvYields']['hp'],
        'tvSTA': resposta_json['tvYields']['sta'],
        'tvSPD': resposta_json['tvYields']['spd'],
        'tvATK': resposta_json['tvYields']['atk'],
        'tvDEF': resposta_json['tvYields']['def'],
        'tvSPATK': resposta_json['tvYields']['spatk'],
        'tvSPDEF': resposta_json['tvYields']['spdef']
    }
    # Temos que verificar se um temtem tem dois tipos para adicionar
    dict_info_temtems['type 1'] = resposta_json['types'][0]
    if len(resposta_json['types']) == 2:
        dict_info_temtems['type 2'] = resposta_json['types'][1]
    else:
        dict_info_temtems['type 2'] = None

    # Verifica se ele tem evolução
    if resposta_json['evolution']['evolves'] == False:
        dict_info_temtems['evolution'] = False
    else:
        dict_info_temtems['evolution'] = True
    return dict_info_temtems
```


```python
# Criando loop para pegar informações de todos os temtem
lista_temtems = []
for numero_temtem in range(1, 165):
    endereco = f"https://temtem-api.mael.tech/api/temtems/{numero_temtem}"
    resposta = requests.get(endereco)
    temp_dict = cria_dict(resposta.json())
    lista_temtems.append(temp_dict)
```


```python
len(lista_temtems)
```




    164




```python
lista_temtems[77]
```




    {'number': 78,
     'name': 'Cycrox',
     'hp': 73,
     'sta': 62,
     'spd': 48,
     'atk': 54,
     'def': 61,
     'spatk': 75,
     'spdef': 71,
     'total': 444,
     'height_cm': 132,
     'weight_kg': 41,
     'trait 1': 'Neurotoxins',
     'trait 2': 'Water Synthesizer',
     'male': 20,
     'female': 80,
     'catchRate': 90,
     'hatchMins': 27,
     'tvHP': 0,
     'tvSTA': 0,
     'tvSPD': 0,
     'tvATK': 0,
     'tvDEF': 0,
     'tvSPATK': 2,
     'tvSPDEF': 0,
     'type 1': 'Digital',
     'type 2': 'Toxic',
     'evolution': False}




```python
# com isso temos uma lista com todos os temtems
# Vamos criar a tabela com todas as informações
df_temtems = pd.DataFrame(data=lista_temtems, columns=lista_temtems[0].keys())
```


```python
df_temtems
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>number</th>
      <th>name</th>
      <th>hp</th>
      <th>sta</th>
      <th>spd</th>
      <th>atk</th>
      <th>def</th>
      <th>spatk</th>
      <th>spdef</th>
      <th>total</th>
      <th>...</th>
      <th>tvHP</th>
      <th>tvSTA</th>
      <th>tvSPD</th>
      <th>tvATK</th>
      <th>tvDEF</th>
      <th>tvSPATK</th>
      <th>tvSPDEF</th>
      <th>type 1</th>
      <th>type 2</th>
      <th>evolution</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>Mimit</td>
      <td>55</td>
      <td>55</td>
      <td>55</td>
      <td>55</td>
      <td>65</td>
      <td>55</td>
      <td>65</td>
      <td>405</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>Digital</td>
      <td>None</td>
      <td>False</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>Oree</td>
      <td>61</td>
      <td>74</td>
      <td>35</td>
      <td>65</td>
      <td>44</td>
      <td>32</td>
      <td>31</td>
      <td>342</td>
      <td>...</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>Digital</td>
      <td>None</td>
      <td>True</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>Zaobian</td>
      <td>72</td>
      <td>90</td>
      <td>51</td>
      <td>84</td>
      <td>50</td>
      <td>42</td>
      <td>44</td>
      <td>433</td>
      <td>...</td>
      <td>0</td>
      <td>2</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>Digital</td>
      <td>None</td>
      <td>True</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>Chromeon</td>
      <td>61</td>
      <td>55</td>
      <td>66</td>
      <td>65</td>
      <td>49</td>
      <td>78</td>
      <td>63</td>
      <td>437</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>2</td>
      <td>0</td>
      <td>Digital</td>
      <td>None</td>
      <td>False</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>Halzhi</td>
      <td>52</td>
      <td>35</td>
      <td>38</td>
      <td>56</td>
      <td>48</td>
      <td>39</td>
      <td>44</td>
      <td>312</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>Digital</td>
      <td>None</td>
      <td>True</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>159</th>
      <td>160</td>
      <td>Monkko</td>
      <td>84</td>
      <td>55</td>
      <td>59</td>
      <td>43</td>
      <td>71</td>
      <td>84</td>
      <td>48</td>
      <td>444</td>
      <td>...</td>
      <td>2</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>2</td>
      <td>0</td>
      <td>Digital</td>
      <td>Melee</td>
      <td>True</td>
    </tr>
    <tr>
      <th>160</th>
      <td>161</td>
      <td>Anahir</td>
      <td>54</td>
      <td>36</td>
      <td>31</td>
      <td>50</td>
      <td>101</td>
      <td>50</td>
      <td>101</td>
      <td>423</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>2</td>
      <td>0</td>
      <td>2</td>
      <td>Crystal</td>
      <td>Fire</td>
      <td>True</td>
    </tr>
    <tr>
      <th>161</th>
      <td>162</td>
      <td>Anatan</td>
      <td>62</td>
      <td>48</td>
      <td>32</td>
      <td>50</td>
      <td>103</td>
      <td>50</td>
      <td>103</td>
      <td>448</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>3</td>
      <td>0</td>
      <td>2</td>
      <td>Crystal</td>
      <td>Fire</td>
      <td>True</td>
    </tr>
    <tr>
      <th>162</th>
      <td>163</td>
      <td>Tyranak</td>
      <td>78</td>
      <td>48</td>
      <td>34</td>
      <td>81</td>
      <td>74</td>
      <td>57</td>
      <td>76</td>
      <td>448</td>
      <td>...</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>Fire</td>
      <td>Nature</td>
      <td>False</td>
    </tr>
    <tr>
      <th>163</th>
      <td>164</td>
      <td>Volgon</td>
      <td>60</td>
      <td>38</td>
      <td>60</td>
      <td>58</td>
      <td>62</td>
      <td>90</td>
      <td>67</td>
      <td>435</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>4</td>
      <td>0</td>
      <td>Electric</td>
      <td>None</td>
      <td>False</td>
    </tr>
  </tbody>
</table>
<p>164 rows × 28 columns</p>
</div>




```python
# FINALMENTE TEMOS UMA TABELA :D
# Agora podemos fazer analise de dados
# Achar os temtems que tem o maior valor de vida base
df_temtems[df_temtems['hp'] == df_temtems['hp'].max()]
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>number</th>
      <th>name</th>
      <th>hp</th>
      <th>sta</th>
      <th>spd</th>
      <th>atk</th>
      <th>def</th>
      <th>spatk</th>
      <th>spdef</th>
      <th>total</th>
      <th>...</th>
      <th>tvHP</th>
      <th>tvSTA</th>
      <th>tvSPD</th>
      <th>tvATK</th>
      <th>tvDEF</th>
      <th>tvSPATK</th>
      <th>tvSPDEF</th>
      <th>type 1</th>
      <th>type 2</th>
      <th>evolution</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>107</th>
      <td>108</td>
      <td>Goolder</td>
      <td>125</td>
      <td>70</td>
      <td>10</td>
      <td>64</td>
      <td>62</td>
      <td>68</td>
      <td>62</td>
      <td>461</td>
      <td>...</td>
      <td>4</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>Toxic</td>
      <td>None</td>
      <td>True</td>
    </tr>
  </tbody>
</table>
<p>1 rows × 28 columns</p>
</div>



# Analise inicial
- Podemos notar que o Goolder tem o maior valor base de vida
- Com isso podemo achar todos os temtem que tem os maiores valores de cada atributo


```python
# Top HP
# df_temtems[df_temtems['hp'] == df_temtems['hp'].max()]

# Top 5 hp
df_temtems.nlargest(5, 'hp')
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>number</th>
      <th>name</th>
      <th>hp</th>
      <th>sta</th>
      <th>spd</th>
      <th>atk</th>
      <th>def</th>
      <th>spatk</th>
      <th>spdef</th>
      <th>total</th>
      <th>...</th>
      <th>tvHP</th>
      <th>tvSTA</th>
      <th>tvSPD</th>
      <th>tvATK</th>
      <th>tvDEF</th>
      <th>tvSPATK</th>
      <th>tvSPDEF</th>
      <th>type 1</th>
      <th>type 2</th>
      <th>evolution</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>107</th>
      <td>108</td>
      <td>Goolder</td>
      <td>125</td>
      <td>70</td>
      <td>10</td>
      <td>64</td>
      <td>62</td>
      <td>68</td>
      <td>62</td>
      <td>461</td>
      <td>...</td>
      <td>4</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>Toxic</td>
      <td>None</td>
      <td>True</td>
    </tr>
    <tr>
      <th>133</th>
      <td>134</td>
      <td>Turoc</td>
      <td>105</td>
      <td>48</td>
      <td>60</td>
      <td>75</td>
      <td>68</td>
      <td>47</td>
      <td>45</td>
      <td>448</td>
      <td>...</td>
      <td>4</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>Wind</td>
      <td>Earth</td>
      <td>True</td>
    </tr>
    <tr>
      <th>147</th>
      <td>148</td>
      <td>Mawmense</td>
      <td>100</td>
      <td>52</td>
      <td>41</td>
      <td>42</td>
      <td>52</td>
      <td>56</td>
      <td>88</td>
      <td>431</td>
      <td>...</td>
      <td>3</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>Digital</td>
      <td>Nature</td>
      <td>True</td>
    </tr>
    <tr>
      <th>68</th>
      <td>69</td>
      <td>Saipat</td>
      <td>92</td>
      <td>42</td>
      <td>70</td>
      <td>80</td>
      <td>55</td>
      <td>70</td>
      <td>50</td>
      <td>459</td>
      <td>...</td>
      <td>3</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>Water</td>
      <td>Melee</td>
      <td>False</td>
    </tr>
    <tr>
      <th>145</th>
      <td>146</td>
      <td>Waspeen</td>
      <td>92</td>
      <td>64</td>
      <td>36</td>
      <td>58</td>
      <td>80</td>
      <td>50</td>
      <td>70</td>
      <td>450</td>
      <td>...</td>
      <td>3</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>2</td>
      <td>0</td>
      <td>0</td>
      <td>Digital</td>
      <td>Crystal</td>
      <td>True</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 28 columns</p>
</div>




```python
# Stamina
df_temtems[df_temtems['sta'] == df_temtems['sta'].max()]
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>number</th>
      <th>name</th>
      <th>hp</th>
      <th>sta</th>
      <th>spd</th>
      <th>atk</th>
      <th>def</th>
      <th>spatk</th>
      <th>spdef</th>
      <th>total</th>
      <th>...</th>
      <th>tvHP</th>
      <th>tvSTA</th>
      <th>tvSPD</th>
      <th>tvATK</th>
      <th>tvDEF</th>
      <th>tvSPATK</th>
      <th>tvSPDEF</th>
      <th>type 1</th>
      <th>type 2</th>
      <th>evolution</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>44</th>
      <td>45</td>
      <td>Babawa</td>
      <td>85</td>
      <td>92</td>
      <td>40</td>
      <td>79</td>
      <td>60</td>
      <td>51</td>
      <td>52</td>
      <td>459</td>
      <td>...</td>
      <td>0</td>
      <td>4</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>Nature</td>
      <td>Water</td>
      <td>True</td>
    </tr>
  </tbody>
</table>
<p>1 rows × 28 columns</p>
</div>




```python
# Speed
df_temtems[df_temtems['spd'] == df_temtems['spd'].max()]
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>number</th>
      <th>name</th>
      <th>hp</th>
      <th>sta</th>
      <th>spd</th>
      <th>atk</th>
      <th>def</th>
      <th>spatk</th>
      <th>spdef</th>
      <th>total</th>
      <th>...</th>
      <th>tvHP</th>
      <th>tvSTA</th>
      <th>tvSPD</th>
      <th>tvATK</th>
      <th>tvDEF</th>
      <th>tvSPATK</th>
      <th>tvSPDEF</th>
      <th>type 1</th>
      <th>type 2</th>
      <th>evolution</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>19</th>
      <td>20</td>
      <td>Amphatyr</td>
      <td>65</td>
      <td>42</td>
      <td>110</td>
      <td>51</td>
      <td>53</td>
      <td>62</td>
      <td>64</td>
      <td>447</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>4</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>Electric</td>
      <td>Nature</td>
      <td>True</td>
    </tr>
  </tbody>
</table>
<p>1 rows × 28 columns</p>
</div>




```python
# attack
df_temtems[df_temtems['atk'] == df_temtems['atk'].max()]
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>number</th>
      <th>name</th>
      <th>hp</th>
      <th>sta</th>
      <th>spd</th>
      <th>atk</th>
      <th>def</th>
      <th>spatk</th>
      <th>spdef</th>
      <th>total</th>
      <th>...</th>
      <th>tvHP</th>
      <th>tvSTA</th>
      <th>tvSPD</th>
      <th>tvATK</th>
      <th>tvDEF</th>
      <th>tvSPATK</th>
      <th>tvSPDEF</th>
      <th>type 1</th>
      <th>type 2</th>
      <th>evolution</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>67</th>
      <td>68</td>
      <td>Osukai</td>
      <td>71</td>
      <td>48</td>
      <td>68</td>
      <td>95</td>
      <td>85</td>
      <td>31</td>
      <td>46</td>
      <td>444</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>4</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>Earth</td>
      <td>Melee</td>
      <td>True</td>
    </tr>
  </tbody>
</table>
<p>1 rows × 28 columns</p>
</div>




```python
# defense
df_temtems[df_temtems['def'] == df_temtems['def'].max()]
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>number</th>
      <th>name</th>
      <th>hp</th>
      <th>sta</th>
      <th>spd</th>
      <th>atk</th>
      <th>def</th>
      <th>spatk</th>
      <th>spdef</th>
      <th>total</th>
      <th>...</th>
      <th>tvHP</th>
      <th>tvSTA</th>
      <th>tvSPD</th>
      <th>tvATK</th>
      <th>tvDEF</th>
      <th>tvSPATK</th>
      <th>tvSPDEF</th>
      <th>type 1</th>
      <th>type 2</th>
      <th>evolution</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>132</th>
      <td>133</td>
      <td>Tuvine</td>
      <td>58</td>
      <td>47</td>
      <td>70</td>
      <td>78</td>
      <td>111</td>
      <td>60</td>
      <td>47</td>
      <td>471</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>4</td>
      <td>0</td>
      <td>0</td>
      <td>Wind</td>
      <td>Crystal</td>
      <td>True</td>
    </tr>
  </tbody>
</table>
<p>1 rows × 28 columns</p>
</div>




```python
# Special attack
df_temtems[df_temtems['spatk'] == df_temtems['spatk'].max()]
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>number</th>
      <th>name</th>
      <th>hp</th>
      <th>sta</th>
      <th>spd</th>
      <th>atk</th>
      <th>def</th>
      <th>spatk</th>
      <th>spdef</th>
      <th>total</th>
      <th>...</th>
      <th>tvHP</th>
      <th>tvSTA</th>
      <th>tvSPD</th>
      <th>tvATK</th>
      <th>tvDEF</th>
      <th>tvSPATK</th>
      <th>tvSPDEF</th>
      <th>type 1</th>
      <th>type 2</th>
      <th>evolution</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>114</th>
      <td>115</td>
      <td>Oceara</td>
      <td>64</td>
      <td>42</td>
      <td>100</td>
      <td>54</td>
      <td>51</td>
      <td>105</td>
      <td>65</td>
      <td>481</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>2</td>
      <td>0</td>
      <td>Water</td>
      <td>None</td>
      <td>False</td>
    </tr>
  </tbody>
</table>
<p>1 rows × 28 columns</p>
</div>




```python
# Special defense
df_temtems[df_temtems['spdef'] == df_temtems['spdef'].max()]
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>number</th>
      <th>name</th>
      <th>hp</th>
      <th>sta</th>
      <th>spd</th>
      <th>atk</th>
      <th>def</th>
      <th>spatk</th>
      <th>spdef</th>
      <th>total</th>
      <th>...</th>
      <th>tvHP</th>
      <th>tvSTA</th>
      <th>tvSPD</th>
      <th>tvATK</th>
      <th>tvDEF</th>
      <th>tvSPATK</th>
      <th>tvSPDEF</th>
      <th>type 1</th>
      <th>type 2</th>
      <th>evolution</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>128</th>
      <td>129</td>
      <td>Adoroboros</td>
      <td>65</td>
      <td>66</td>
      <td>60</td>
      <td>29</td>
      <td>42</td>
      <td>70</td>
      <td>110</td>
      <td>442</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>4</td>
      <td>Toxic</td>
      <td>Mental</td>
      <td>False</td>
    </tr>
  </tbody>
</table>
<p>1 rows × 28 columns</p>
</div>




```python
# Total master Temtem
df_temtems[df_temtems['total'] == df_temtems['total'].max()]
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>number</th>
      <th>name</th>
      <th>hp</th>
      <th>sta</th>
      <th>spd</th>
      <th>atk</th>
      <th>def</th>
      <th>spatk</th>
      <th>spdef</th>
      <th>total</th>
      <th>...</th>
      <th>tvHP</th>
      <th>tvSTA</th>
      <th>tvSPD</th>
      <th>tvATK</th>
      <th>tvDEF</th>
      <th>tvSPATK</th>
      <th>tvSPDEF</th>
      <th>type 1</th>
      <th>type 2</th>
      <th>evolution</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>11</th>
      <td>12</td>
      <td>Tateru</td>
      <td>79</td>
      <td>85</td>
      <td>60</td>
      <td>78</td>
      <td>66</td>
      <td>54</td>
      <td>66</td>
      <td>488</td>
      <td>...</td>
      <td>0</td>
      <td>2</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>Neutral</td>
      <td>None</td>
      <td>False</td>
    </tr>
  </tbody>
</table>
<p>1 rows × 28 columns</p>
</div>



# Não esperado!
- Não esperava que TATERU tivesse o maior total de pontos de atributos 

# Podemos buscar todos os temtems do tipo crystal
- Para isso basca usar o atributo type 1 e typ2 um dos dois tem que ser igual a crystal


```python
df_temtems[(df_temtems['type 1'] == 'Crystal')  | (df_temtems['type 2'] == 'Crystal')][['number', 'name', 'type 1', 'type 2']]
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>number</th>
      <th>name</th>
      <th>type 1</th>
      <th>type 2</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>20</th>
      <td>21</td>
      <td>Bunbun</td>
      <td>Earth</td>
      <td>Crystal</td>
    </tr>
    <tr>
      <th>21</th>
      <td>22</td>
      <td>Mudrid</td>
      <td>Earth</td>
      <td>Crystal</td>
    </tr>
    <tr>
      <th>38</th>
      <td>39</td>
      <td>Lapinite</td>
      <td>Crystal</td>
      <td>None</td>
    </tr>
    <tr>
      <th>39</th>
      <td>40</td>
      <td>Azuroc</td>
      <td>Crystal</td>
      <td>None</td>
    </tr>
    <tr>
      <th>40</th>
      <td>41</td>
      <td>Zenoreth</td>
      <td>Crystal</td>
      <td>None</td>
    </tr>
    <tr>
      <th>49</th>
      <td>50</td>
      <td>Valash</td>
      <td>Neutral</td>
      <td>Crystal</td>
    </tr>
    <tr>
      <th>53</th>
      <td>54</td>
      <td>Gyalis</td>
      <td>Crystal</td>
      <td>Melee</td>
    </tr>
    <tr>
      <th>54</th>
      <td>55</td>
      <td>Occlura</td>
      <td>Crystal</td>
      <td>None</td>
    </tr>
    <tr>
      <th>55</th>
      <td>56</td>
      <td>Myx</td>
      <td>Crystal</td>
      <td>Mental</td>
    </tr>
    <tr>
      <th>71</th>
      <td>72</td>
      <td>Crystle</td>
      <td>Crystal</td>
      <td>None</td>
    </tr>
    <tr>
      <th>72</th>
      <td>73</td>
      <td>Sherald</td>
      <td>Crystal</td>
      <td>None</td>
    </tr>
    <tr>
      <th>73</th>
      <td>74</td>
      <td>Tortenite</td>
      <td>Crystal</td>
      <td>Toxic</td>
    </tr>
    <tr>
      <th>74</th>
      <td>75</td>
      <td>Innki</td>
      <td>Electric</td>
      <td>Crystal</td>
    </tr>
    <tr>
      <th>121</th>
      <td>122</td>
      <td>Shuine</td>
      <td>Crystal</td>
      <td>Water</td>
    </tr>
    <tr>
      <th>132</th>
      <td>133</td>
      <td>Tuvine</td>
      <td>Wind</td>
      <td>Crystal</td>
    </tr>
    <tr>
      <th>145</th>
      <td>146</td>
      <td>Waspeen</td>
      <td>Digital</td>
      <td>Crystal</td>
    </tr>
    <tr>
      <th>156</th>
      <td>157</td>
      <td>Chimurian</td>
      <td>Nature</td>
      <td>Crystal</td>
    </tr>
    <tr>
      <th>160</th>
      <td>161</td>
      <td>Anahir</td>
      <td>Crystal</td>
      <td>Fire</td>
    </tr>
    <tr>
      <th>161</th>
      <td>162</td>
      <td>Anatan</td>
      <td>Crystal</td>
      <td>Fire</td>
    </tr>
  </tbody>
</table>
</div>



# Deixar a tabela melhor
- Como cada temtem já tem um número identificador podemos usar esse número como índice


```python
df_temtem_com_indice = df_temtems.set_index('number')
```


```python
df_temtem_com_indice
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>name</th>
      <th>hp</th>
      <th>sta</th>
      <th>spd</th>
      <th>atk</th>
      <th>def</th>
      <th>spatk</th>
      <th>spdef</th>
      <th>total</th>
      <th>height_cm</th>
      <th>...</th>
      <th>tvHP</th>
      <th>tvSTA</th>
      <th>tvSPD</th>
      <th>tvATK</th>
      <th>tvDEF</th>
      <th>tvSPATK</th>
      <th>tvSPDEF</th>
      <th>type 1</th>
      <th>type 2</th>
      <th>evolution</th>
    </tr>
    <tr>
      <th>number</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
      <td>Mimit</td>
      <td>55</td>
      <td>55</td>
      <td>55</td>
      <td>55</td>
      <td>65</td>
      <td>55</td>
      <td>65</td>
      <td>405</td>
      <td>42</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>Digital</td>
      <td>None</td>
      <td>False</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Oree</td>
      <td>61</td>
      <td>74</td>
      <td>35</td>
      <td>65</td>
      <td>44</td>
      <td>32</td>
      <td>31</td>
      <td>342</td>
      <td>128</td>
      <td>...</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>Digital</td>
      <td>None</td>
      <td>True</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Zaobian</td>
      <td>72</td>
      <td>90</td>
      <td>51</td>
      <td>84</td>
      <td>50</td>
      <td>42</td>
      <td>44</td>
      <td>433</td>
      <td>213</td>
      <td>...</td>
      <td>0</td>
      <td>2</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>Digital</td>
      <td>None</td>
      <td>True</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Chromeon</td>
      <td>61</td>
      <td>55</td>
      <td>66</td>
      <td>65</td>
      <td>49</td>
      <td>78</td>
      <td>63</td>
      <td>437</td>
      <td>77</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>2</td>
      <td>0</td>
      <td>Digital</td>
      <td>None</td>
      <td>False</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Halzhi</td>
      <td>52</td>
      <td>35</td>
      <td>38</td>
      <td>56</td>
      <td>48</td>
      <td>39</td>
      <td>44</td>
      <td>312</td>
      <td>90</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>Digital</td>
      <td>None</td>
      <td>True</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>160</th>
      <td>Monkko</td>
      <td>84</td>
      <td>55</td>
      <td>59</td>
      <td>43</td>
      <td>71</td>
      <td>84</td>
      <td>48</td>
      <td>444</td>
      <td>158</td>
      <td>...</td>
      <td>2</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>2</td>
      <td>0</td>
      <td>Digital</td>
      <td>Melee</td>
      <td>True</td>
    </tr>
    <tr>
      <th>161</th>
      <td>Anahir</td>
      <td>54</td>
      <td>36</td>
      <td>31</td>
      <td>50</td>
      <td>101</td>
      <td>50</td>
      <td>101</td>
      <td>423</td>
      <td>152</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>2</td>
      <td>0</td>
      <td>2</td>
      <td>Crystal</td>
      <td>Fire</td>
      <td>True</td>
    </tr>
    <tr>
      <th>162</th>
      <td>Anatan</td>
      <td>62</td>
      <td>48</td>
      <td>32</td>
      <td>50</td>
      <td>103</td>
      <td>50</td>
      <td>103</td>
      <td>448</td>
      <td>220</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>3</td>
      <td>0</td>
      <td>2</td>
      <td>Crystal</td>
      <td>Fire</td>
      <td>True</td>
    </tr>
    <tr>
      <th>163</th>
      <td>Tyranak</td>
      <td>78</td>
      <td>48</td>
      <td>34</td>
      <td>81</td>
      <td>74</td>
      <td>57</td>
      <td>76</td>
      <td>448</td>
      <td>420</td>
      <td>...</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>Fire</td>
      <td>Nature</td>
      <td>False</td>
    </tr>
    <tr>
      <th>164</th>
      <td>Volgon</td>
      <td>60</td>
      <td>38</td>
      <td>60</td>
      <td>58</td>
      <td>62</td>
      <td>90</td>
      <td>67</td>
      <td>435</td>
      <td>1200</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>4</td>
      <td>0</td>
      <td>Electric</td>
      <td>None</td>
      <td>False</td>
    </tr>
  </tbody>
</table>
<p>164 rows × 27 columns</p>
</div>




```python
df_temtem_com_indice.loc[164]
```




    name                    Volgon
    hp                          60
    sta                         38
    spd                         60
    atk                         58
    def                         62
    spatk                       90
    spdef                       67
    total                      435
    height_cm                 1200
    weight_kg                 6200
    trait 1          Short Circuit
    trait 2      Superconductivity
    male                        50
    female                      50
    catchRate                    0
    hatchMins                 45.0
    tvHP                         0
    tvSTA                        0
    tvSPD                        0
    tvATK                        0
    tvDEF                        0
    tvSPATK                      4
    tvSPDEF                      0
    type 1                Electric
    type 2                    None
    evolution                False
    Name: 164, dtype: object



# Vamos salvar nossa tabela agora como csv
- Vamos salvar nossa tabela para csv para utilizar em Power BI e fazer mais análise de forma mais fácil


```python
df_temtem_com_indice.to_csv('tabela_temtem.csv')
```


```python
# Podemos importar o arquivo para ver se ta tudo bem
teste_tabela_csv = pd.read_csv('tabela_temtem.csv', index_col="number")
teste_tabela_csv
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>name</th>
      <th>hp</th>
      <th>sta</th>
      <th>spd</th>
      <th>atk</th>
      <th>def</th>
      <th>spatk</th>
      <th>spdef</th>
      <th>total</th>
      <th>height_cm</th>
      <th>...</th>
      <th>tvHP</th>
      <th>tvSTA</th>
      <th>tvSPD</th>
      <th>tvATK</th>
      <th>tvDEF</th>
      <th>tvSPATK</th>
      <th>tvSPDEF</th>
      <th>type 1</th>
      <th>type 2</th>
      <th>evolution</th>
    </tr>
    <tr>
      <th>number</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
      <td>Mimit</td>
      <td>55</td>
      <td>55</td>
      <td>55</td>
      <td>55</td>
      <td>65</td>
      <td>55</td>
      <td>65</td>
      <td>405</td>
      <td>42</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>Digital</td>
      <td>NaN</td>
      <td>False</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Oree</td>
      <td>61</td>
      <td>74</td>
      <td>35</td>
      <td>65</td>
      <td>44</td>
      <td>32</td>
      <td>31</td>
      <td>342</td>
      <td>128</td>
      <td>...</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>Digital</td>
      <td>NaN</td>
      <td>True</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Zaobian</td>
      <td>72</td>
      <td>90</td>
      <td>51</td>
      <td>84</td>
      <td>50</td>
      <td>42</td>
      <td>44</td>
      <td>433</td>
      <td>213</td>
      <td>...</td>
      <td>0</td>
      <td>2</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>Digital</td>
      <td>NaN</td>
      <td>True</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Chromeon</td>
      <td>61</td>
      <td>55</td>
      <td>66</td>
      <td>65</td>
      <td>49</td>
      <td>78</td>
      <td>63</td>
      <td>437</td>
      <td>77</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>2</td>
      <td>0</td>
      <td>Digital</td>
      <td>NaN</td>
      <td>False</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Halzhi</td>
      <td>52</td>
      <td>35</td>
      <td>38</td>
      <td>56</td>
      <td>48</td>
      <td>39</td>
      <td>44</td>
      <td>312</td>
      <td>90</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>Digital</td>
      <td>NaN</td>
      <td>True</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>160</th>
      <td>Monkko</td>
      <td>84</td>
      <td>55</td>
      <td>59</td>
      <td>43</td>
      <td>71</td>
      <td>84</td>
      <td>48</td>
      <td>444</td>
      <td>158</td>
      <td>...</td>
      <td>2</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>2</td>
      <td>0</td>
      <td>Digital</td>
      <td>Melee</td>
      <td>True</td>
    </tr>
    <tr>
      <th>161</th>
      <td>Anahir</td>
      <td>54</td>
      <td>36</td>
      <td>31</td>
      <td>50</td>
      <td>101</td>
      <td>50</td>
      <td>101</td>
      <td>423</td>
      <td>152</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>2</td>
      <td>0</td>
      <td>2</td>
      <td>Crystal</td>
      <td>Fire</td>
      <td>True</td>
    </tr>
    <tr>
      <th>162</th>
      <td>Anatan</td>
      <td>62</td>
      <td>48</td>
      <td>32</td>
      <td>50</td>
      <td>103</td>
      <td>50</td>
      <td>103</td>
      <td>448</td>
      <td>220</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>3</td>
      <td>0</td>
      <td>2</td>
      <td>Crystal</td>
      <td>Fire</td>
      <td>True</td>
    </tr>
    <tr>
      <th>163</th>
      <td>Tyranak</td>
      <td>78</td>
      <td>48</td>
      <td>34</td>
      <td>81</td>
      <td>74</td>
      <td>57</td>
      <td>76</td>
      <td>448</td>
      <td>420</td>
      <td>...</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>Fire</td>
      <td>Nature</td>
      <td>False</td>
    </tr>
    <tr>
      <th>164</th>
      <td>Volgon</td>
      <td>60</td>
      <td>38</td>
      <td>60</td>
      <td>58</td>
      <td>62</td>
      <td>90</td>
      <td>67</td>
      <td>435</td>
      <td>1200</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>4</td>
      <td>0</td>
      <td>Electric</td>
      <td>NaN</td>
      <td>False</td>
    </tr>
  </tbody>
</table>
<p>164 rows × 27 columns</p>
</div>




```python
# Aplicando uma função para encontrar os valores médios de cada tipo
df_temtem_com_indice['type 1'].unique()
```




    array(['Digital', 'Water', 'Nature', 'Neutral', 'Toxic', 'Melee', 'Wind',
           'Electric', 'Earth', 'Mental', 'Fire', 'Crystal'], dtype=object)




```python
df_avg_temtem_type = pd.DataFrame(data=df_temtem_com_indice['type 1'].unique(), columns=['Type'])
```


```python
df_avg_temtem_type
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Type</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Digital</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Water</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Nature</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Neutral</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Toxic</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Melee</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Wind</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Electric</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Earth</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Mental</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Fire</td>
    </tr>
    <tr>
      <th>11</th>
      <td>Crystal</td>
    </tr>
  </tbody>
</table>
</div>




```python
np.average(df_temtem_com_indice[(df_temtem_com_indice['type 1'] == 'Electric') | (df_temtem_com_indice['type 2'] == 'Electric')]['atk'])
```




    62.4




```python
df_avg_temtem_type['Avg_atk'] = [0] * len(df_avg_temtem_type)
for type_of_temtem in df_avg_temtem_type['Type']:
    df_avg_temtem_type.loc[df_avg_temtem_type['Type'] == type_of_temtem, 'Avg_atk'] = np.average(df_temtem_com_indice[(df_temtem_com_indice['type 1'] == type_of_temtem) | (df_temtem_com_indice['type 2'] == type_of_temtem)]['atk'])
```


```python
df_avg_temtem_type
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Type</th>
      <th>Avg_atk</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Digital</td>
      <td>55.380952</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Water</td>
      <td>59.227273</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Nature</td>
      <td>56.583333</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Neutral</td>
      <td>66.384615</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Toxic</td>
      <td>59.550000</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Melee</td>
      <td>69.333333</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Wind</td>
      <td>58.952381</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Electric</td>
      <td>62.400000</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Earth</td>
      <td>66.681818</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Mental</td>
      <td>61.000000</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Fire</td>
      <td>60.590909</td>
    </tr>
    <tr>
      <th>11</th>
      <td>Crystal</td>
      <td>60.473684</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Vamos criar um loop para pegar a media de cada valor importante
# Lista dos tipos de valores que queremos pegar a média
avg_types = ['hp', 'sta', 'spd', 'atk', 'def', 'spatk', 'spdef', 'total', 'height_cm', 'weight_kg']
```


```python
# Loop para criar a média de cada tipo de valor por tipo de temtem
for status_type in avg_types:
    for type_of_temtem in df_avg_temtem_type['Type']:
        df_avg_temtem_type.loc[df_avg_temtem_type['Type'] == type_of_temtem, f'Avg_{status_type}'] = np.average(df_temtem_com_indice[(df_temtem_com_indice['type 1'] == type_of_temtem) | (df_temtem_com_indice['type 2'] == type_of_temtem)][status_type])
```


```python
df_avg_temtem_type
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Type</th>
      <th>Avg_atk</th>
      <th>Avg_hp</th>
      <th>Avg_sta</th>
      <th>Avg_spd</th>
      <th>Avg_def</th>
      <th>Avg_spatk</th>
      <th>Avg_spdef</th>
      <th>Avg_total</th>
      <th>Avg_height_cm</th>
      <th>Avg_weight_kg</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Digital</td>
      <td>55.380952</td>
      <td>64.428571</td>
      <td>54.190476</td>
      <td>55.000000</td>
      <td>56.476190</td>
      <td>55.904762</td>
      <td>56.666667</td>
      <td>398.047619</td>
      <td>116.619048</td>
      <td>70.333333</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Water</td>
      <td>59.227273</td>
      <td>65.681818</td>
      <td>57.590909</td>
      <td>61.272727</td>
      <td>54.954545</td>
      <td>68.227273</td>
      <td>57.318182</td>
      <td>424.272727</td>
      <td>145.681818</td>
      <td>87.318182</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Nature</td>
      <td>56.583333</td>
      <td>65.083333</td>
      <td>58.625000</td>
      <td>54.916667</td>
      <td>57.041667</td>
      <td>53.083333</td>
      <td>62.625000</td>
      <td>407.958333</td>
      <td>144.666667</td>
      <td>273.250000</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Neutral</td>
      <td>66.384615</td>
      <td>68.846154</td>
      <td>59.769231</td>
      <td>60.692308</td>
      <td>65.230769</td>
      <td>59.384615</td>
      <td>60.923077</td>
      <td>441.230769</td>
      <td>156.153846</td>
      <td>115.076923</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Toxic</td>
      <td>59.550000</td>
      <td>67.550000</td>
      <td>49.600000</td>
      <td>59.600000</td>
      <td>58.000000</td>
      <td>62.600000</td>
      <td>59.550000</td>
      <td>416.450000</td>
      <td>153.300000</td>
      <td>205.500000</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Melee</td>
      <td>69.333333</td>
      <td>66.416667</td>
      <td>53.583333</td>
      <td>66.083333</td>
      <td>62.583333</td>
      <td>51.875000</td>
      <td>51.541667</td>
      <td>421.416667</td>
      <td>160.750000</td>
      <td>130.916667</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Wind</td>
      <td>58.952381</td>
      <td>61.238095</td>
      <td>52.095238</td>
      <td>66.095238</td>
      <td>57.809524</td>
      <td>61.666667</td>
      <td>64.809524</td>
      <td>422.666667</td>
      <td>133.714286</td>
      <td>55.571429</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Electric</td>
      <td>62.400000</td>
      <td>60.000000</td>
      <td>54.450000</td>
      <td>64.400000</td>
      <td>52.550000</td>
      <td>54.100000</td>
      <td>57.500000</td>
      <td>405.400000</td>
      <td>190.400000</td>
      <td>412.750000</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Earth</td>
      <td>66.681818</td>
      <td>68.681818</td>
      <td>51.727273</td>
      <td>57.045455</td>
      <td>64.545455</td>
      <td>51.000000</td>
      <td>49.772727</td>
      <td>409.454545</td>
      <td>140.863636</td>
      <td>106.500000</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Mental</td>
      <td>61.000000</td>
      <td>56.000000</td>
      <td>55.857143</td>
      <td>63.904762</td>
      <td>50.619048</td>
      <td>63.523810</td>
      <td>62.428571</td>
      <td>413.333333</td>
      <td>143.190476</td>
      <td>67.095238</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Fire</td>
      <td>60.590909</td>
      <td>63.136364</td>
      <td>50.772727</td>
      <td>57.000000</td>
      <td>64.863636</td>
      <td>60.409091</td>
      <td>55.772727</td>
      <td>412.545455</td>
      <td>162.590909</td>
      <td>353.818182</td>
    </tr>
    <tr>
      <th>11</th>
      <td>Crystal</td>
      <td>60.473684</td>
      <td>63.526316</td>
      <td>49.684211</td>
      <td>61.052632</td>
      <td>68.578947</td>
      <td>58.736842</td>
      <td>64.157895</td>
      <td>426.210526</td>
      <td>169.157895</td>
      <td>121.736842</td>
    </tr>
  </tbody>
</table>
</div>



- Com isso criamos uma tabela com os valores das médias por tipo de temtem


```python
import matplotlib.pyplot as plt
import seaborn as sns
```


```python
plt.figure(figsize=(10,8))
sns.barplot(x=df_avg_temtem_type['Type'], y=df_avg_temtem_type['Avg_weight_kg'])
```




    <AxesSubplot:xlabel='Type', ylabel='Avg_weight_kg'>




    
![png](Temtem_Infos_40_1.png)
    



```python
# Adicionando total de temtem por tipos
for type_of_temtem in df_avg_temtem_type['Type']:
    qtd_temtem = len(df_temtem_com_indice[(df_temtem_com_indice['type 1'] == type_of_temtem) | (df_temtem_com_indice['type 2'] == type_of_temtem)])
    df_avg_temtem_type.loc[df_avg_temtem_type['Type'] == type_of_temtem, 'Total_Count'] = qtd_temtem
```


```python
df_avg_temtem_type
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Type</th>
      <th>Avg_atk</th>
      <th>Avg_hp</th>
      <th>Avg_sta</th>
      <th>Avg_spd</th>
      <th>Avg_def</th>
      <th>Avg_spatk</th>
      <th>Avg_spdef</th>
      <th>Avg_total</th>
      <th>Avg_height_cm</th>
      <th>Avg_weight_kg</th>
      <th>Total_Count</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Digital</td>
      <td>55.380952</td>
      <td>64.428571</td>
      <td>54.190476</td>
      <td>55.000000</td>
      <td>56.476190</td>
      <td>55.904762</td>
      <td>56.666667</td>
      <td>398.047619</td>
      <td>116.619048</td>
      <td>70.333333</td>
      <td>21.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Water</td>
      <td>59.227273</td>
      <td>65.681818</td>
      <td>57.590909</td>
      <td>61.272727</td>
      <td>54.954545</td>
      <td>68.227273</td>
      <td>57.318182</td>
      <td>424.272727</td>
      <td>145.681818</td>
      <td>87.318182</td>
      <td>22.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Nature</td>
      <td>56.583333</td>
      <td>65.083333</td>
      <td>58.625000</td>
      <td>54.916667</td>
      <td>57.041667</td>
      <td>53.083333</td>
      <td>62.625000</td>
      <td>407.958333</td>
      <td>144.666667</td>
      <td>273.250000</td>
      <td>24.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Neutral</td>
      <td>66.384615</td>
      <td>68.846154</td>
      <td>59.769231</td>
      <td>60.692308</td>
      <td>65.230769</td>
      <td>59.384615</td>
      <td>60.923077</td>
      <td>441.230769</td>
      <td>156.153846</td>
      <td>115.076923</td>
      <td>13.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Toxic</td>
      <td>59.550000</td>
      <td>67.550000</td>
      <td>49.600000</td>
      <td>59.600000</td>
      <td>58.000000</td>
      <td>62.600000</td>
      <td>59.550000</td>
      <td>416.450000</td>
      <td>153.300000</td>
      <td>205.500000</td>
      <td>20.0</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Melee</td>
      <td>69.333333</td>
      <td>66.416667</td>
      <td>53.583333</td>
      <td>66.083333</td>
      <td>62.583333</td>
      <td>51.875000</td>
      <td>51.541667</td>
      <td>421.416667</td>
      <td>160.750000</td>
      <td>130.916667</td>
      <td>24.0</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Wind</td>
      <td>58.952381</td>
      <td>61.238095</td>
      <td>52.095238</td>
      <td>66.095238</td>
      <td>57.809524</td>
      <td>61.666667</td>
      <td>64.809524</td>
      <td>422.666667</td>
      <td>133.714286</td>
      <td>55.571429</td>
      <td>21.0</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Electric</td>
      <td>62.400000</td>
      <td>60.000000</td>
      <td>54.450000</td>
      <td>64.400000</td>
      <td>52.550000</td>
      <td>54.100000</td>
      <td>57.500000</td>
      <td>405.400000</td>
      <td>190.400000</td>
      <td>412.750000</td>
      <td>20.0</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Earth</td>
      <td>66.681818</td>
      <td>68.681818</td>
      <td>51.727273</td>
      <td>57.045455</td>
      <td>64.545455</td>
      <td>51.000000</td>
      <td>49.772727</td>
      <td>409.454545</td>
      <td>140.863636</td>
      <td>106.500000</td>
      <td>22.0</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Mental</td>
      <td>61.000000</td>
      <td>56.000000</td>
      <td>55.857143</td>
      <td>63.904762</td>
      <td>50.619048</td>
      <td>63.523810</td>
      <td>62.428571</td>
      <td>413.333333</td>
      <td>143.190476</td>
      <td>67.095238</td>
      <td>21.0</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Fire</td>
      <td>60.590909</td>
      <td>63.136364</td>
      <td>50.772727</td>
      <td>57.000000</td>
      <td>64.863636</td>
      <td>60.409091</td>
      <td>55.772727</td>
      <td>412.545455</td>
      <td>162.590909</td>
      <td>353.818182</td>
      <td>22.0</td>
    </tr>
    <tr>
      <th>11</th>
      <td>Crystal</td>
      <td>60.473684</td>
      <td>63.526316</td>
      <td>49.684211</td>
      <td>61.052632</td>
      <td>68.578947</td>
      <td>58.736842</td>
      <td>64.157895</td>
      <td>426.210526</td>
      <td>169.157895</td>
      <td>121.736842</td>
      <td>19.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_avg_temtem_type.loc[df_avg_temtem_type['Type'] == 'Digital']
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Type</th>
      <th>Avg_atk</th>
      <th>Avg_hp</th>
      <th>Avg_sta</th>
      <th>Avg_spd</th>
      <th>Avg_def</th>
      <th>Avg_spatk</th>
      <th>Avg_spdef</th>
      <th>Avg_total</th>
      <th>Avg_height_cm</th>
      <th>Avg_weight_kg</th>
      <th>Total_Count</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Digital</td>
      <td>55.380952</td>
      <td>64.428571</td>
      <td>54.190476</td>
      <td>55.0</td>
      <td>56.47619</td>
      <td>55.904762</td>
      <td>56.666667</td>
      <td>398.047619</td>
      <td>116.619048</td>
      <td>70.333333</td>
      <td>21.0</td>
    </tr>
  </tbody>
</table>
</div>



# Fim das análises por enquanto
