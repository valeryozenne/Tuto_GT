# Lecture 3 : Basic reconstruction using Python

Title : Basic reconstruction using Python

Schedule : June 11, 2020 | 16:00-17:00 

Speaker: Valéry Ozenne

## Summary

 - [Avant propos](#avant-propos)
 - [Sequence and Data](#sequence-and-data)
 - [Objectives](#objectives)
 - [A typical Python Gadget](#a-typical-python-gadget)
 - [My first Python Gadget](#my-first-python-gadget)
   - [Folder structure](#folder-structure)
   - [Writing the gadget](#writing-the-gadget)
   - [Compilation and installation](#compilation-and-installation)
   - [Reconstruction and visualisation](#demandes-de-fusion)
   - [Exercice 1 : find the number of readout](#exercice -1 : find-the-number-of-readout)
   - [Exercice 2 : display the matrix size](#demandes-de-fusion)
   - [Exercice 3 : display the AcquisitionHeader](#demandes-de-fusion)
   - [Exercice 4 : readout selection and corresponding encoding space](#demandes-de-fusion)
   - [Exercice 5 : Buffering](#buffering)
   - [Exercice 6 : Fourier Transform](#exercice-6-:-fourier-transform)
   - [First conclusion](#first-conclusion)
 - [A brief description of the class used to store readout, kspace or image data](#)
    - [Readout](#les-issues)
    - [Kspace](#faq)
    - [Image](#liens)
 - [My First Data Buffered Gadget](#glossaire)
    - [Writing the Buffered Gadget](#les-issues)
    - [Compilation, Installation and launching the reco](#faq)
    - [Exercice 1: Fourier Transform using imsmrmrd-python-tool](#liens)
    - [Exercice 2: Fourier Transform using BART](#liens)
    - [Exercice 3: Fourier Transform using Sigpy](#liens)
    - [Exercice 4: Grappa reconstruction using PyGrappa](#liens)

## Avant propos

Pourquoi les pythons gadgets. 

Gadgetron : atout majeur : recosntruction en ligne , intégrable sur les machines Siemens et GE. Facilité de prototypage et de déploiement pour des études multi-centriques , imagerie interventionnelle. 


## Sequence and Data

We will use acquisitions from the SMS sequence of the CMRR: https://www.cmrr.umn.edu/multiband/ acquired on a 3T Prisma from Siemens.

Data is available at this [link](https://zenodo.org/):

- acquisition single-shot grandient EPI, 12 slices, 3 repetitions, in-plane acceleration none, slice acceleration none   
- acquisition single-shot grandient EPI, 12 slices, 3 repetitions, in-plane acceleration 2, slice acceleration none  
- acquisition single-shot grandient EPI, 12 slices, 3 repetitions, in-plane acceleration 2, slice acceleration 2 

The data has been converted with siemens_to_ismrmrd, we will not discuss data conversion here. This will be the object of the following readings.

## Objectives

- to become familiar with the Cartesian reconstruction pipeline
- to create new python gadget from scratch
- to create a new xml configuration file 
- data manipulation (readout, kspace, image)
- to call BART from a python gagdet
- to call SigPy from a python gagdet


## A typical Python Gadget


```python
import sys
import ismrmrd
import ismrmrd.xsd

from gadgetron import Gadget

class MyFirstPythonGadget(Gadget):
    def __init__(self,next_gadget=None):
        super(MyFirstPythonGadget,self).__init__(next_gadget=next_gadget)
        pass        

    def process_config(self, conf):
        # do allocation        
	pass 

    def process(self, message):
               
	# get information from the message

	# modify the message

	# send the message to the next gadget
        self.put_next(message)
        return 0

```

* __init__ 

    I quote Google: "**__init__** is a reseved method in python classes. It is called as a constructor in object oriented terminology. 
    This method is called when an object is created from a class and it allows the class to initialize the attributes of the class."

* process_config()
 
    This function is called only once at the start and is generally used for allocation and initialization.
    Function responsible for reading the **FlexibleHeader** (**ISMRMRDHeader**) which contains general information specific to the acquisition and therefore identical whatever the readouts
    

* process()
    
    Function responsible for receiving all messages from the previous gadget and for sending a new message to the next gadget.
    It may or may not interact with the information contained in the message.
 

## My first Python Gadget

### Folder structure

Nous allons modifier les sources du Gadgetron et ajouter à la fois des nouveaux gadgets et des nouveaux pipelines de reconstruction qui appelleront ces gadgets.

Le repertoire de travail est le suivant: ${GT_SOURCE_FOLDER}/gadgets/python/legacy/ qui contient un dossier `config/` avec les fichies .xml et un dossier `gadgets/` avec les gadgets python 

```bash

├── CMakeLists.txt
├── config
│   └── Generic_Cartesian_Grappa_RealTimeCine_Python.xml
└── legacy
    ├── config
    │   ├── pseudoreplica.xml
    │   ├── python_buckets.xml
    │   ├── python_image_array_recon.xml
    │   ├── python_image_array.xml
    │   ├── python_passthrough.xml
    │   └── python_short.xml
    └── gadgets
        ├── accumulate_and_recon.py
        ├── array_image.py
        ├── bucket_recon.py
        ├── image_array_recon.py
        ├── image_array_recon_rtcine_plotting.py
        ├── passthrough_array_image.py
        ├── passthrough.py
        ├── pseudoreplicagather.py
        ├── remove_2x_oversampling.py
        └── rms_coil_combine.py
```



### Writing the gadget

Dans `${GT_SOURCE_FOLDER}/gadgets/python/legacy/gadgets/`, créer le fichier my_first_python_gadget.py puis copier la classe précédente.
Il est cependant nécessaire de modifier un élément qui est le message, ici on reçoit en fait deux messages en même temps. 
* Un header appelé **AcquisitionHeader** et noté **header** qui contient en autre les informations d'encodage spatial 
* Des données sous forme matricielle en complex float notée **data** , la dimension de matrice est [RO, CHA] (readout size, channel number)
 
```
process(self, message):
#devient
process(self, head, data):
```

```
self.put_next(message):
#devient
self.put_next(head, data):
```

```
On peut ajouter le message suivant dans la fonction process pour vérifier que l'on passe bien dedans
print("so far, so good")
```

### Compilation and installation

Nous allons premièrement créer un nouveau fichier xml nommé python_passthrough_tutorial.xml.

```

<?xml version="1.0" encoding="utf-8"?>
<gadgetronStreamConfiguration xsi:schemaLocation="http://gadgetron.sf.net/gadgetron gadgetron.xsd"
        xmlns="http://gadgetron.sf.net/gadgetron"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">

    <!--
        Gadgetron generic recon chain for 2D and 3D cartesian sampling
        
    -->

    <!-- reader -->
    <reader><slot>1008</slot><dll>gadgetron_mricore</dll><classname>GadgetIsmrmrdAcquisitionMessageReader</classname></reader>
    <reader><slot>1026</slot><dll>gadgetron_mricore</dll><classname>GadgetIsmrmrdWaveformMessageReader</classname></reader>

    <!-- writer -->
    <writer><slot>1022</slot><dll>gadgetron_mricore</dll><classname>MRIImageWriter</classname></writer>

    


</gadgetronStreamConfiguration>
```


Pour appeler notre gadget, nous devons l'ajouter à la chaine de reconstruction qui est actuellement vide. Pour cela ajouter les lignes suivantes après le ``MRIImageWriter``

```
<gadget>
        <name>MyFirstPythonGadget</name>
        <dll>gadgetron_python</dll>
        <classname>PythonGadget</classname>
        <property><name>python_path</name><value>/home/myuser/scripts/python</value></property>
        <property><name>python_module</name><value>my_first_python_gadget</value></property>
        <property><name>python_class</name><value>MyFirstPythonGadget</value></property>
</gadget>
```

Dans le CMakeLists.txt, dans la partie `set(gadgetron_python_config_files` ajouter la ligne : `config/python_passthrough_tutorial.xml`

et ajouter dans la partie `set(gadgetron_python_gadgets_files`, ajouter lal ligne suviante: `gadgets/my_first_python_gadget.py`

Il nous faut maintenant compiler le Gadgetron


```bash
cmake ../
make 
sudo make install
gadgetron
```

### Reconstruction and visualisation

Lancer la commande suivante pour effectuer la reconstruction après avoir redémarrer le Gadgetron

```
rm out.h5
gadgetron_ismrmrd_client -f     -c python_passthrough_tutorial.xml

```

Les données sont écrites par défaut dans un fichier out.h5.

Utiliser le script python pour les lire et afficher les images

```

```


### Exercice 1 : find the number of readout

Ajouter les lignes suivantes dans la fonction process.

```
self.counter=self.counter+1;
print(self.counter)
```

Puis rejouer l'étape compilation et installation et lancer la reco.

### Exercice 2 : display the matrix size

Ajouter les lignes suivantes dans la fonction process et import numpy as np en entête

```
print(np.shape(data))
```

Puis rejouer l'étape compilation et installation et lancer la reco.

### Exercice 3 : display the AcquisitionHeader

Ajouter les lignes suivantes

print(header)


Nous allons obtenir ce type de message

```
2404
(256, 12)
version: 1
flags: 2097152
measurement_uid: 26
scan_counter: 2404
acquisition_time_stamp: 22684240
physiology_time_stamp: 8545183, 0, 0
number_of_samples: 256
available_channels: 12
active_channels: 12
channel_mask: 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0
discard_pre: 0
discard_post: 0
center_sample: 128
encoding_space_ref: 0
trajectory_dimensions: 0
sample_time_us: 1.600000023841858
position: 0.0, -4.842615127563477, -75.69005584716797
read_dir: -6.123031769111886e-17, 1.0, 0.0
phase_dir: 1.0, 6.123031769111886e-17, 0.0
slice_dir: 0.0, 0.0, 1.0
patient_table_position: 0.0, 0.0, -1259016.0
idx: kspace_encode_step_1: 23
kspace_encode_step_2: 0
average: 0
slice: 0
contrast: 0
phase: 0
repetition: 2
set: 0
segment: 1
user: 0, 0, 0, 0, 0, 32, 0, 0

user_int: 0, 0, 0, 0, 0, 0, 0, 0
user_float: 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0
```

### Exercice 4 : readout selection and corresponding encoding space

Récupérer le numero de repetition et de slice avec les lignes suivantes:

```
slice = header.idx.slice
repetition=  header.idx.repetition
e1=header.idx.kspace_encode_step_1
segment=header.idx.segment
```

Et ajouter la condition suivante pour ne pas transmettre tous les readouts des autres coupes et autres répétitions.

```
if (repetition>0 and slice>0):
   print(self.counter_send, " slice: ",slice , " rep: ", repetition, " e1: ", e1," segment: ",  segment)
    
```

Ajouter un second compteur nommé self.counter_send qui s'incrémente dans cette boucle.



### Exercice 5 : Buffering

self.myBuffer = None

self.enc = None

Dans process config ajouter ceci pour récupérer la taille de la amtrice:

self.header = ismrmrd.xsd.CreateFromDocument(conf)
self.enc = self.header.encoding[0]


```
if self.myBuffer is None:
     channels = acq.active_channels
            
     eNz = self.enc.encodedSpace.matrixSize.z
     eNy = self.enc.encodedSpace.matrixSize.y
     eNx = self.enc.encodedSpace.matrixSize.x
        
     self.myBuffer = np.zeros(( int(eNx),eNy,channels),dtype=np.complex64)

self.myBuffer[:,e1,:] = data
```

### Exercice 6 : Fourier Transform

```
from matplotlib import transform
from ismrmrdtools import transform


if (e1==96)
    plt.figure(1)    
    plt.imshow(np.abs(self.myBuffer[:,:,0]))
``` 

### First Conclusion

Pas forcément nécessaire de tout redévelopper

Il existe de nombreux gadgets en python et/ou en C++ qui permettent d'aller plus vite.

 
TODO: installation sans GPU / avec GPU
      docker avec/sans Python / Matlab












## A brief description of the class used to store readout, kspace or image data

The data structures in the gadgetron vary during reconstruction. It is important to differenciate, the class or common structures 

* used to store a unit of readout that would feed into a buffer
* used to store a unit of data that would feed into a reconstruction
* used to store an array of reconstructed data

Each of them are defined in a C++ and have equivalent in Python. Additionnal structure are also present and you can create new one

### Readout

Le gadget python recevra deux messages associés qui contiennent le **AcquisitionHeader** et les données sous forme de matrice **hoNDArray< std::complex<float> >**.
En python le **hoNDArray** est la matrice multidimensionnel **ndarray** issue de la librairie numpy 

``` 
process(self, header, data):

```

```
print(type(header))

print(type(data))
```

### Kspace

En imagerie cartesienne, deux gadgets jouent un role fondamental : AcquisitionAccumulateTriggerGadget et BucketToBufferGadget. 

Ces gadgets servent à bufferiser les readouts afin de construire le kspace. En IRM, les dimensionalités sont très nombreuses: 

* kx (RO)
* ky (E1)
* kz (E2)
* channels (CHA)
* average
* repetition
* segment
* contrast
* phase
* set
* slice (SLC)
* ...

Par convention, nous aurons en entrée des matrices de dimentions [RO, CHA] et en sortie de BucketToBufferGadget des matrices de dimensions [RO E1 E2 CHA N S SLC].
Les dimensions **N** et **S** peuvent être choisie arbitrairement par l'utilisateur. 

Il est fort interessant de se positionner après ces gadgets ou sont automatiquement triés les données kspaces, que ce soit les lignes de calibration en imagerie parallèle ou les lignes sous échantionnées.  


Les données de calibration si elles sont présentes sont accessibles via la structure suivante:

```
buffer.ref.data  
buffer.ref.header
```

Les données "standard" sont accessibles via la structure suivante:

```
buffer.data.data
buffer.data.header
```

Attention, la taille des headers est associée la taille des données, les headers sont généralement différents, par ex la position des coupes changent suivant la direction SLC.
Nous avons donc maintenant une matrice hondarray de acquisitionheader de dimensions [E1 E2 N S SLC]. Les headers étant identique suivant la direction de readout et pour tous les éléments d'antennes.

 


Le gadget python correspondant recevra un message qui contient une structure nommé **IsmrmrdReconBit** nommée IsmrmrdReconBit et IsmrmrdDataBuffered. Le gadget python doit donc contenir les includes suivants.   

``` 
from gadgetron import Gadget,IsmrmrdDataBuffered, IsmrmrdReconBit, SamplingLimit,SamplingDescription, IsmrmrdImageArray


process(self, buffer):

```

Pour aller plus loin, voici les classes C++ correspondantes.



```
struct IsmrmrdReconBit
  {
  public:
    IsmrmrdDataBuffered data_;
    boost::optional<IsmrmrdDataBuffered> ref_;
  }

```

```
struct IsmrmrdDataBuffered
  {
  public:
    //7D, fixed order [E0, E1, E2, CHA, N, S, LOC]
    hoNDArray< std::complex<float> > data_;
    
    //7D, fixed order [TRAJ, E0, E1, E2, N, S, LOC]
    boost::optional<hoNDArray<float>> trajectory_;

    // 6D, density weights [E0, E1, E2, N, S, LOC]
    boost::optional<hoNDArray<float> > density_;

    //5D, fixed order [E1, E2, N, S, LOC]
    hoNDArray< ISMRMRD::AcquisitionHeader > headers_;

    SamplingDescription sampling_;
   }

```


### Image


### Writing the Buffered Gadget

Nous allons donc maintenant créer un nouveau gadget nommé MyFirstDataBufferedGadget.



```python
import sys
import ismrmrd
import ismrmrd.xsd

from gadgetron import Gadget,IsmrmrdDataBuffered, IsmrmrdReconBit, SamplingLimit,SamplingDescription, IsmrmrdImageArray

class MyFirstDataBufferedGadget(Gadget):
    def __init__(self,next_gadget=None):
        super(MyFirstDataBufferedGadget,self).__init__(next_gadget=next_gadget)
        self.my_value = 0
        self.my_list = []
	self.my_matrix =[]

    def process_config(self, conf):
        # do allocation        
	pass 

    def process(self, message):
               
	# get information from the message

	# modify the message

	# send the message to the next gadget
        self.put_next(message)
        return 0

```


### Compilation, Installation and launching the reco
### Exercice 1: Fourier Transform using imsmrmrd-python-tool
### [Exercice 2: Fourier Transform using BART
### Exercice 3: Fourier Transform using Sigpy
### Exercice 4: Grappa reconstruction using PyGrappa




