

```python
import requests
import json
import pandas as pd
import urllib2
import random
import numpy as np
from bs4 import BeautifulSoup
from pathlib import Path
```

# Renseignement du numéro d'épisode


```python
# quel est le numero de l'épisode a telecharger
numero_episode = str(402407)
url_get = 'https://services.radio-canada.ca/neuro/v1/episodes/' + numero_episode
```

# Obtention des informations de l'émission parente de l'épisode


```python
# Obtention des informations de l'émission 

# Interrogation API 
emission = requests.get(url_get,
             headers={'Authorization':'Client-Key bf9ac6d8-9ad8-4124-a63c-7b7bdf22a2ee'})

contenu_emission = emission.json()


```


```python
# Obtention du nom et de l'ID de l'émission

value1 = pd.DataFrame(contenu_emission.get('ancestors'))

id_emission = str(value1.iloc[0,2])
title_emission = value1.iloc[0,4]

# Obtention de l'adresse du podcast à télécharger

value2 = pd.DataFrame(contenu_emission.get('podcastItem'))
lien_url_podcast = str(value2.iloc[1,5])

lien_url_podcast

lien_url_podcast = lien_url_podcast.split("{u'href': u'",1)[1] 

lien_url_podcast = lien_url_podcast.split(".mp3", 1)[0]

lien = lien_url_podcast + '.mp3'


```


```python
#  Telechargement du fichier mp3

import urllib2
# response = urllib2.urlopen(lien)
# html = response.read()



file_name = lien.split('/')[-1]
u = urllib2.urlopen(lien)
f = open(file_name, 'wb')
meta = u.info()
file_size = int(meta.getheaders("Content-Length")[0])
print "Downloading: %s Bytes: %s" % (file_name, file_size)



```

    Downloading: 2018-03-10_14_35_14_laspherebalado_0000.mp3 Bytes: 84499436


# Obtention des différents segments de l'épisode


```python
# obtention de l'information relative à l'emission et à ses différentes séquences  dans l'API

response =  requests.get(url_get + '/clips',
             headers={'Authorization':'Client-Key bf9ac6d8-9ad8-4124-a63c-7b7bdf22a2ee'})

contenu = response.json()


```


```python
# for contentType in contenu:
#         print contentType['id'],contentType['title'],contentType['durationInSeconds'], contentType['summary']
#        try:
#            print contentType['summaryMultimediaItem']['summaryImage']['concreteImages'][0]['mediaLink']['href']
#        except Exception:
#            print 'No image'
```

## Constitution d'une table avec les informations nécessaires au découpage et à l'application

On calcule aussi la seconde de début et de fin de chaque segment.
Par prudence, on exporte aussi cette table en fichier csv.



```python
case_list = []
for contentType in contenu:
    image_url = None
    try:
        image_url = contentType['summaryMultimediaItem']['summaryImage']['concreteImages'][0]['mediaLink']['href']
    except Exception:
        pass
    case = [contentType['id'], contentType['title'], contentType['durationInSeconds'], contentType['broadcastedFirstTimeAt'], image_url, contentType['summary']]
    case_list.append(case)
        
df = pd.DataFrame(case_list)

df.columns = ['id_section', 'title', 'durationInSeconds',  'broadcastedFirstTimeAt', 'imageUrl', 'resume']


df = df[df.title.str.contains("Bulletin") == False]

df =  df.reset_index(drop=True)


df['fin_seq']=pd.to_numeric(df.durationInSeconds, errors='ignore').cumsum()
df['deb_seq']=df['fin_seq']-pd.to_numeric(df.durationInSeconds, errors='ignore')
df['deb_seq']=df['deb_seq']+1
df.columns = ['id_section', 'title', 'durationInSeconds', 'broadcastedFirstTimeAt', 'imageUrl',  'resume', 'fin_seq', 'debut_seq']
df['title_emission']=title_emission
df['id_emission']=id_emission
df['id_episode']=numero_episode




number = min(df.count())

value_na = random.sample(range(5000, 9000), 9)


m = df['id_section'].isnull()
#count rows with NaNs
l = m.sum()
#create array with size l
s = np.random.choice(value_na, size=l)
#set NaNs values
df.loc[m, 'id_section'] = s

date_episode=min(df['broadcastedFirstTimeAt'])
date_episode



file_name_csv = '/Users/usr/Google_Drive/Hack/Enregistrement/'+str(emission)+'_'+str(numero_episode)+'_'+str(date_episode)+".csv"
df.to_csv(file_name_csv, sep=';', encoding='utf-8')

df
#for i in range(0:df.ncount)""


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
      <th>id_section</th>
      <th>title</th>
      <th>durationInSeconds</th>
      <th>broadcastedFirstTimeAt</th>
      <th>imageUrl</th>
      <th>resume</th>
      <th>fin_seq</th>
      <th>debut_seq</th>
      <th>title_emission</th>
      <th>id_emission</th>
      <th>id_episode</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>8082</td>
      <td>Ouverture de l'émission spéciale South by Sout...</td>
      <td>370</td>
      <td>2018-03-10T18:06:00.000Z</td>
      <td>http://img.radio-canada.ca/ouglo/emission/635x...</td>
      <td></td>
      <td>370</td>
      <td>1</td>
      <td>La sphère</td>
      <td>377</td>
      <td>402407</td>
    </tr>
    <tr>
      <th>1</th>
      <td>62826</td>
      <td>Stefanka, une solution à l'absence de standard...</td>
      <td>738</td>
      <td>2018-03-10T18:12:10.000Z</td>
      <td>https://images.radio-canada.ca/v1/ici-info/1x1...</td>
      <td>&lt;p&gt;Le Québec compte sur une grande délégation ...</td>
      <td>1108</td>
      <td>371</td>
      <td>La sphère</td>
      <td>377</td>
      <td>402407</td>
    </tr>
    <tr>
      <th>2</th>
      <td>8541</td>
      <td>Entrevue avec Paul Kruszewski, fondateur et PD...</td>
      <td>582</td>
      <td>2018-03-10T18:24:28.000Z</td>
      <td>http://img.radio-canada.ca/ouglo/emission/635x...</td>
      <td></td>
      <td>1690</td>
      <td>1109</td>
      <td>La sphère</td>
      <td>377</td>
      <td>402407</td>
    </tr>
    <tr>
      <th>3</th>
      <td>62831</td>
      <td>Capital de risque&amp;nbsp;: transformer de jeunes...</td>
      <td>508</td>
      <td>2018-03-10T18:34:10.000Z</td>
      <td>https://images.radio-canada.ca/v1/ici-info/1x1...</td>
      <td>&lt;p&gt;L'univers des jeunes pousses carbure au ris...</td>
      <td>2198</td>
      <td>1691</td>
      <td>La sphère</td>
      <td>377</td>
      <td>402407</td>
    </tr>
    <tr>
      <th>4</th>
      <td>6424</td>
      <td>Chronique de Catherine Mathys&amp;nbsp;:&amp;nbsp;Impr...</td>
      <td>441</td>
      <td>2018-03-10T18:42:38.000Z</td>
      <td>http://img.radio-canada.ca/ouglo/emission/635x...</td>
      <td></td>
      <td>2639</td>
      <td>2199</td>
      <td>La sphère</td>
      <td>377</td>
      <td>402407</td>
    </tr>
    <tr>
      <th>5</th>
      <td>8082</td>
      <td>Chronique de Fabien Loszach&amp;nbsp;:&amp;nbsp;Retour...</td>
      <td>556</td>
      <td>2018-03-10T18:49:59.000Z</td>
      <td>http://img.radio-canada.ca/ouglo/emission/635x...</td>
      <td></td>
      <td>3195</td>
      <td>2640</td>
      <td>La sphère</td>
      <td>377</td>
      <td>402407</td>
    </tr>
  </tbody>
</table>
</div>



# Déclinaison de la table pour alimenter le script de découpage


```python
df1 = pd.DataFrame(df)
df1 = df1[['debut_seq','fin_seq', 'id_section']]


# dfList = df1.tolist()

# df['deb_seq']=df['fin_seq'] - df['durationInSeconds']
df2 = df1.values.tolist()
# 

df2

# print('\n'.join(' '.join(sub) for sub in df2))
# test = ["Jargon", "Hello", "This", "Is", "Great"]
# group = '\n'.join([df2[0]] + ['{'+item+'}' for item in df2[1:]])
# print(group)

# dfList
```




    [[1, 370, 8082],
     [371, 1108, u'62826'],
     [1109, 1690, 8541],
     [1691, 2198, u'62831'],
     [2199, 2639, 6424],
     [2640, 3195, 8082]]




```python
joined = '\n'.join(' '.join(map(str, row)) for row in df2)
joined
```




    '1 370 8082\n371 1108 62826\n1109 1690 8541\n1691 2198 62831\n2199 2639 6424\n2640 3195 8082'



# Découpage des segments


```python
from audioclipextractor import AudioClipExtractor, SpecsParser


# Inicialize the extractor
ext = AudioClipExtractor(file_name, '/usr/local/bin/ffmpeg')

# Define the clips to extract
# It's possible to pass a file instead of a string
specs = '\n' + joined + '\n '
specs

# Extract the clips according to the specs and save them as a zip archive
ext.extractClips(specs, '/Users/usr/Google_Drive/Hack/Enregistrement/', zipOutput=False)


```

# Changement des noms des fichiers pour mieux les identifier


```python
total_rows = df.count()
end_loop = max(total_rows)+1
```




    7924




```python
import os
for x in xrange(0, end_loop):
    my_file = Path("/Users/usr/Google_Drive/Hack/Enregistrement/clip01.mp3")
    if my_file.is_file():
        if x <9:
            old_file_name = 'clip0'+str(x+1)+'.mp3'
            section= df['id_section'][x]
            episode= df['id_episode'][x]
            emission = df['id_emission'][x]
            new_file_name = str(emission)+'_'+str(episode)+'_'+str(section)+'.mp3'
            old_file = os.path.join("/Users/usr/Google_Drive/Hack/Enregistrement/", old_file_name)
            new_file = os.path.join("/Users/usr/Google_Drive/Hack/Enregistrement/", new_file_name)
            print (old_file)
            print(new_file)
            os.rename(old_file, new_file)
        elif x==9:
            old_file_name = 'clip10'+'.mp3'
            section= df['id_section'][x]
            episode= df['id_episode'][x]
            emission = df['id_emission'][x]
            new_file_name = str(emission)+'_'+str(episode)+'_'+str(section)+'.mp3'
            old_file = os.path.join("/Users/usr/Google_Drive/Hack/Enregistrement/", old_file_name)
            new_file = os.path.join("/Users/usr/Google_Drive/Hack/Enregistrement/", new_file_name)
            print (old_file)
            print(new_file)
            os.rename(old_file, new_file)        
        else:
            old_file_name = 'clip'+str(x+1)+'.mp3'
            section= df['id_section'][x]
            episode= df['id_episode'][x]
            emission = df['id_emission'][x]
            new_file_name = str(emission)+'_'+str(episode)+'_'+str(section)+'.mp3'
            old_file = os.path.join("/Users/usr/Google_Drive/Hack/Enregistrement/", old_file_name)
            new_file = os.path.join("/Users/usr/Google_Drive/Hack/Enregistrement/", new_file_name)
            os.rename(old_file, new_file)
    else:
        old_file_name = 'clip'+str(x+1)+'.mp3'
        section= df['id_section'][x]
        episode= df['id_episode'][x]
        emission = df['id_emission'][x]
        new_file_name = str(emission)+'_'+str(episode)+'_'+str(section)+'.mp3'
        old_file = os.path.join("/Users/usr/Google_Drive/Hack/Enregistrement/", old_file_name)
        new_file = os.path.join("/Users/usr/Google_Drive/Hack/Enregistrement/", new_file_name)
        os.rename(old_file, new_file)
        
    
    
    



```

    /Users/usr/Google_Drive/Hack/Enregistrement/clip1.mp3
    /Users/usr/Google_Drive/Hack/Enregistrement/377_402407_7924.mp3
    /Users/usr/Google_Drive/Hack/Enregistrement/clip2.mp3
    /Users/usr/Google_Drive/Hack/Enregistrement/377_402407_62826.mp3
    /Users/usr/Google_Drive/Hack/Enregistrement/clip3.mp3
    /Users/usr/Google_Drive/Hack/Enregistrement/377_402407_5210.mp3
    /Users/usr/Google_Drive/Hack/Enregistrement/clip4.mp3
    /Users/usr/Google_Drive/Hack/Enregistrement/377_402407_62831.mp3
    /Users/usr/Google_Drive/Hack/Enregistrement/clip5.mp3
    /Users/usr/Google_Drive/Hack/Enregistrement/377_402407_8620.mp3
    /Users/usr/Google_Drive/Hack/Enregistrement/clip6.mp3
    /Users/usr/Google_Drive/Hack/Enregistrement/377_402407_7924.mp3



    ---------------------------------------------------------------------------

    KeyError                                  Traceback (most recent call last)

    <ipython-input-41-0ba528d3c950> in <module>()
         38     else:
         39         old_file_name = 'clip'+str(x+1)+'.mp3'
    ---> 40         section= df['id_section'][x]
         41         episode= df['id_episode'][x]
         42         emission = df['id_emission'][x]


    /anaconda2/lib/python2.7/site-packages/pandas/core/series.pyc in __getitem__(self, key)
        621         key = com._apply_if_callable(key, self)
        622         try:
    --> 623             result = self.index.get_value(self, key)
        624 
        625             if not is_scalar(result):


    /anaconda2/lib/python2.7/site-packages/pandas/core/indexes/base.pyc in get_value(self, series, key)
       2558         try:
       2559             return self._engine.get_value(s, k,
    -> 2560                                           tz=getattr(series.dtype, 'tz', None))
       2561         except KeyError as e1:
       2562             if len(self) > 0 and self.inferred_type in ['integer', 'boolean']:


    pandas/_libs/index.pyx in pandas._libs.index.IndexEngine.get_value()


    pandas/_libs/index.pyx in pandas._libs.index.IndexEngine.get_value()


    pandas/_libs/index.pyx in pandas._libs.index.IndexEngine.get_loc()


    pandas/_libs/hashtable_class_helper.pxi in pandas._libs.hashtable.Int64HashTable.get_item()


    pandas/_libs/hashtable_class_helper.pxi in pandas._libs.hashtable.Int64HashTable.get_item()


    KeyError: 6

