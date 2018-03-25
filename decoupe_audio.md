

```python
import requests
import json
import pandas as pd
import urllib2
import random
import numpy as np
from bs4 import BeautifulSoup
from pathlib import Path
from pydub import AudioSegment
```

# Renseignement du numéro d'épisode


```python
# quel est le numero de l'épisode a telecharger
# 402378
# 403261
# 402688

numero_episode = str(402378)
url_get = 'https://services.radio-canada.ca/neuro/v1/episodes/' + numero_episode
```

# Obtention des informations de l'émission parente de l'épisode


```python
# Obtention des informations de l'émission 

# Interrogation API 
emission = requests.get(url_get,
             headers={'Authorization':'Client-Key xxxx'})

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

lien_url_podcast_titre=lien_url_podcast
lien_url_podcast = lien_url_podcast.split("{u'href': u'",1)[1] 

lien_url_podcast = lien_url_podcast.split(".mp3", 1)[0]

lien = lien_url_podcast + '.mp3'

lien
```




    'http://medias-balado.radio-canada.ca/diffusion/2018/03/balado/src/CBF/2018-03-11_14_00_00_lesanneeslumierebalado_0000.mp3'




```python
#  Telechargement du fichier mp3

import urllib2
#response = urllib2.urlopen(lien)
# html = response.read()

file_name = lien.split('/')[-1]


mp3file = urllib2.urlopen(lien)
with open(file_name,'wb') as output:
  output.write(mp3file.read())











```

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



file_name_csv = 'Enregistrement/Finaux/'+str(id_emission)+'_'+str(numero_episode)+'_'+str(date_episode)+".csv"
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
      <td>5641</td>
      <td>Sommaire de l'émission avec Sophie-Andrée Blon...</td>
      <td>153</td>
      <td>2018-03-11T16:10:00.000Z</td>
      <td>http://img.radio-canada.ca/ouglo/emission/1250...</td>
      <td></td>
      <td>153</td>
      <td>1</td>
      <td>Les années lumière</td>
      <td>52</td>
      <td>402378</td>
    </tr>
    <tr>
      <th>1</th>
      <td>62881</td>
      <td>Élire des scientifiques au Congrès américain?</td>
      <td>797</td>
      <td>2018-03-11T16:12:33.000Z</td>
      <td>https://images.radio-canada.ca/v1/ici-premiere...</td>
      <td>&lt;p&gt;Aux États-Unis, quelque 500 citoyens ayant ...</td>
      <td>950</td>
      <td>154</td>
      <td>Les années lumière</td>
      <td>52</td>
      <td>402378</td>
    </tr>
    <tr>
      <th>2</th>
      <td>8815</td>
      <td>Le son de la science</td>
      <td>130</td>
      <td>2018-03-11T16:25:50.000Z</td>
      <td>http://img.radio-canada.ca/ouglo/emission/1250...</td>
      <td></td>
      <td>1080</td>
      <td>951</td>
      <td>Les années lumière</td>
      <td>52</td>
      <td>402378</td>
    </tr>
    <tr>
      <th>3</th>
      <td>62882</td>
      <td>Plus d’un million de manchots passés sous le r...</td>
      <td>785</td>
      <td>2018-03-11T16:28:00.000Z</td>
      <td>https://images.radio-canada.ca/v1/ici-premiere...</td>
      <td>&lt;p&gt;Les bonnes nouvelles qui viennent de l'Arct...</td>
      <td>1865</td>
      <td>1081</td>
      <td>Les années lumière</td>
      <td>52</td>
      <td>402378</td>
    </tr>
    <tr>
      <th>4</th>
      <td>62883</td>
      <td>Prana au Planétarium</td>
      <td>549</td>
      <td>2018-03-11T16:41:05.000Z</td>
      <td>https://images.radio-canada.ca/v1/ici-premiere...</td>
      <td>&lt;p&gt;Prana souhaite devenir astronome. Cet intér...</td>
      <td>2414</td>
      <td>1866</td>
      <td>Les années lumière</td>
      <td>52</td>
      <td>402378</td>
    </tr>
    <tr>
      <th>5</th>
      <td>62886</td>
      <td>Le clonage se rapproche davantage de l’humain</td>
      <td>541</td>
      <td>2018-03-11T16:50:14.000Z</td>
      <td>https://images.radio-canada.ca/v1/ici-premiere...</td>
      <td>&lt;p&gt;Des biologistes chinois ont récemment effec...</td>
      <td>2955</td>
      <td>2415</td>
      <td>Les années lumière</td>
      <td>52</td>
      <td>402378</td>
    </tr>
    <tr>
      <th>6</th>
      <td>6946</td>
      <td>Sommaire de la 2e heure avec Sophie-Andrée Blo...</td>
      <td>89</td>
      <td>2018-03-11T17:06:00.000Z</td>
      <td>http://img.radio-canada.ca/ouglo/emission/1250...</td>
      <td></td>
      <td>3044</td>
      <td>2956</td>
      <td>Les années lumière</td>
      <td>52</td>
      <td>402378</td>
    </tr>
    <tr>
      <th>7</th>
      <td>6736</td>
      <td>Revue de l'actualité scientifique de la semaine</td>
      <td>540</td>
      <td>2018-03-11T17:07:29.000Z</td>
      <td>http://img.radio-canada.ca/ouglo/emission/1250...</td>
      <td></td>
      <td>3584</td>
      <td>3045</td>
      <td>Les années lumière</td>
      <td>52</td>
      <td>402378</td>
    </tr>
    <tr>
      <th>8</th>
      <td>63063</td>
      <td>Des scientifiques recommandent de classer les ...</td>
      <td>780</td>
      <td>2018-03-11T17:15:00.000Z</td>
      <td>https://images.radio-canada.ca/v1/ici-premiere...</td>
      <td>&lt;p&gt;Le diabète connaît une progression fulguran...</td>
      <td>4364</td>
      <td>3585</td>
      <td>Les années lumière</td>
      <td>52</td>
      <td>402378</td>
    </tr>
    <tr>
      <th>9</th>
      <td>62884</td>
      <td>Pourquoi le big bang a eu lieu à Saint-Eustache</td>
      <td>563</td>
      <td>2018-03-11T17:28:00.000Z</td>
      <td>https://images.radio-canada.ca/v1/ici-premiere...</td>
      <td>&lt;p&gt;Des ovnis à la couleur de vos yeux, nous tr...</td>
      <td>4927</td>
      <td>4365</td>
      <td>Les années lumière</td>
      <td>52</td>
      <td>402378</td>
    </tr>
    <tr>
      <th>10</th>
      <td>62885</td>
      <td>Les pensées secrètes de Picasso</td>
      <td>808</td>
      <td>2018-03-11T17:37:23.000Z</td>
      <td>https://images.radio-canada.ca/v1/ici-premiere...</td>
      <td>&lt;p&gt;De nouvelles analyses physico-chimiques des...</td>
      <td>5735</td>
      <td>4928</td>
      <td>Les années lumière</td>
      <td>52</td>
      <td>402378</td>
    </tr>
    <tr>
      <th>11</th>
      <td>7847</td>
      <td>Le son de la science&amp;nbsp;:&amp;nbsp;La réponse av...</td>
      <td>189</td>
      <td>2018-03-11T17:50:51.000Z</td>
      <td>http://img.radio-canada.ca/ouglo/emission/1250...</td>
      <td></td>
      <td>5924</td>
      <td>5736</td>
      <td>Les années lumière</td>
      <td>52</td>
      <td>402378</td>
    </tr>
    <tr>
      <th>12</th>
      <td>6946</td>
      <td>La science ailleurs avec Découverte</td>
      <td>240</td>
      <td>2018-03-11T17:54:00.000Z</td>
      <td>http://img.radio-canada.ca/ouglo/emission/1250...</td>
      <td></td>
      <td>6164</td>
      <td>5925</td>
      <td>Les années lumière</td>
      <td>52</td>
      <td>402378</td>
    </tr>
    <tr>
      <th>13</th>
      <td>8815</td>
      <td>Mot de la fin</td>
      <td>85</td>
      <td>2018-03-11T17:57:20.000Z</td>
      <td>http://img.radio-canada.ca/ouglo/emission/1250...</td>
      <td></td>
      <td>6249</td>
      <td>6165</td>
      <td>Les années lumière</td>
      <td>52</td>
      <td>402378</td>
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




    [[1, 153, 5641],
     [154, 950, u'62881'],
     [951, 1080, 8815],
     [1081, 1865, u'62882'],
     [1866, 2414, u'62883'],
     [2415, 2955, u'62886'],
     [2956, 3044, 6946],
     [3045, 3584, 6736],
     [3585, 4364, u'63063'],
     [4365, 4927, u'62884'],
     [4928, 5735, u'62885'],
     [5736, 5924, 7847],
     [5925, 6164, 6946],
     [6165, 6249, 8815]]




```python
joined = '\n'.join(' '.join(map(str, row)) for row in df2)
joined
```




    '1 153 5641\n154 950 62881\n951 1080 8815\n1081 1865 62882\n1866 2414 62883\n2415 2955 62886\n2956 3044 6946\n3045 3584 6736\n3585 4364 63063\n4365 4927 62884\n4928 5735 62885\n5736 5924 7847\n5925 6164 6946\n6165 6249 8815'



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
ext.extractClips(specs, '/Users/usr/Google_Drive/Hack/Enregistrement/Finaux', zipOutput=False)


```

# Changement des noms des fichiers pour mieux les identifier et conversion en .wav


```python
total_rows = df.count()
end_loop = max(total_rows)+1
```


```python
boucle=0
import os
for x in xrange(0, end_loop):
    my_file = Path("Enregistrement/Finaux/clip01.mp3")
    if my_file.is_file() | boucle==1:
        boucle=1
        if x <9:
            old_file_name = 'clip0'+str(x+1)+'.mp3'
            section= df['id_section'][x]
            episode= df['id_episode'][x]
            emission = df['id_emission'][x]
            new_file_name = str(emission)+'_'+str(episode)+'_'+str(section)+'.mp3'
            old_file = os.path.join("Enregistrement/Finaux/", old_file_name)
            new_file = os.path.join("Enregistrement/Finaux/", new_file_name)
            print (old_file)
            print(new_file)
            os.rename(old_file, new_file)
            sound = AudioSegment.from_mp3(new_file)
            sound.export(new_file+'.wav', format="wav")
        elif x==9:
            old_file_name = 'clip10'+'.mp3'
            section= df['id_section'][x]
            episode= df['id_episode'][x]
            emission = df['id_emission'][x]
            new_file_name = str(emission)+'_'+str(episode)+'_'+str(section)+'.mp3'
            old_file = os.path.join("Enregistrement/Finaux/", old_file_name)
            new_file = os.path.join("Enregistrement/Finaux/", new_file_name)
            print (old_file)
            print(new_file)
            os.rename(old_file, new_file)
            sound = AudioSegment.from_mp3(new_file)
            sound.export(new_file+'.wav', format="wav")
       
        else:
            old_file_name = 'clip'+str(x+1)+'.mp3'
            section= df['id_section'][x]
            episode= df['id_episode'][x]
            emission = df['id_emission'][x]
            new_file_name = str(emission)+'_'+str(episode)+'_'+str(section)+'.mp3'
            old_file = os.path.join("Enregistrement/Finaux/", old_file_name)
            new_file = os.path.join("Enregistrement/Finaux/", new_file_name)
            os.rename(old_file, new_file)
            sound = AudioSegment.from_mp3(new_file)
            sound.export(new_file+'.wav', format="wav")
    else:
        old_file_name = 'clip'+str(x+1)+'.mp3'
        section= df['id_section'][x]
        episode= df['id_episode'][x]
        emission = df['id_emission'][x]
        new_file_name = str(emission)+'_'+str(episode)+'_'+str(section)+'.mp3'
        old_file = os.path.join("Enregistrement/Finaux/", old_file_name)
        new_file = os.path.join("Enregistrement/Finaux/", new_file_name)
        os.rename(old_file, new_file)
        sound = AudioSegment.from_mp3(new_file)
        sound.export(new_file+'.wav', format="wav")
        
    
    
    



```

    Enregistrement/Finaux/clip01.mp3
    Enregistrement/Finaux/52_402378_5641.mp3
    Enregistrement/Finaux/clip02.mp3
    Enregistrement/Finaux/52_402378_62881.mp3
    Enregistrement/Finaux/clip03.mp3
    Enregistrement/Finaux/52_402378_8815.mp3
    Enregistrement/Finaux/clip04.mp3
    Enregistrement/Finaux/52_402378_62882.mp3
    Enregistrement/Finaux/clip05.mp3
    Enregistrement/Finaux/52_402378_62883.mp3
    Enregistrement/Finaux/clip06.mp3
    Enregistrement/Finaux/52_402378_62886.mp3
    Enregistrement/Finaux/clip07.mp3
    Enregistrement/Finaux/52_402378_6946.mp3
    Enregistrement/Finaux/clip08.mp3
    Enregistrement/Finaux/52_402378_6736.mp3
    Enregistrement/Finaux/clip09.mp3
    Enregistrement/Finaux/52_402378_63063.mp3
    Enregistrement/Finaux/clip10.mp3
    Enregistrement/Finaux/52_402378_62884.mp3



    ---------------------------------------------------------------------------

    CouldntDecodeError                        Traceback (most recent call last)

    <ipython-input-178-228018cdd7d1> in <module>()
         41             new_file = os.path.join("Enregistrement/Finaux/", new_file_name)
         42             os.rename(old_file, new_file)
    ---> 43             sound = AudioSegment.from_mp3(new_file)
         44             sound.export(new_file+'.wav', format="wav")
         45     else:


    /anaconda2/lib/python2.7/site-packages/pydub/audio_segment.pyc in from_mp3(cls, file, parameters)
        530     @classmethod
        531     def from_mp3(cls, file, parameters=None):
    --> 532         return cls.from_file(file, 'mp3', parameters)
        533 
        534     @classmethod


    /anaconda2/lib/python2.7/site-packages/pydub/audio_segment.pyc in from_file(cls, file, format, codec, parameters, **kwargs)
        518         try:
        519             if p.returncode != 0:
    --> 520                 raise CouldntDecodeError("Decoding failed. ffmpeg returned error code: {0}\n\nOutput from ffmpeg/avlib:\n\n{1}".format(p.returncode, p_err))
        521             obj = cls._from_safe_wav(output)
        522         finally:


    CouldntDecodeError: Decoding failed. ffmpeg returned error code: 1
    
    Output from ffmpeg/avlib:
    
    ffmpeg version 3.4.2 Copyright (c) 2000-2018 the FFmpeg developers
      built with Apple LLVM version 9.0.0 (clang-900.0.39.2)
      configuration: --prefix=/usr/local/Cellar/ffmpeg/3.4.2 --enable-shared --enable-pthreads --enable-version3 --enable-hardcoded-tables --enable-avresample --cc=clang --host-cflags= --host-ldflags= --disable-jack --enable-gpl --enable-libmp3lame --enable-libx264 --enable-libxvid --enable-opencl --enable-videotoolbox --disable-lzma
      libavutil      55. 78.100 / 55. 78.100
      libavcodec     57.107.100 / 57.107.100
      libavformat    57. 83.100 / 57. 83.100
      libavdevice    57. 10.100 / 57. 10.100
      libavfilter     6.107.100 /  6.107.100
      libavresample   3.  7.  0 /  3.  7.  0
      libswscale      4.  8.100 /  4.  8.100
      libswresample   2.  9.100 /  2.  9.100
      libpostproc    54.  7.100 / 54.  7.100
    [mp3 @ 0x7f8b3d000000] Failed to read frame size: Could not seek to 1026.
    /var/folders/90/lpdcjhys0lv6hz5y6c9_xy7w0000gn/T/tmpUyxoQJ: Invalid argument


