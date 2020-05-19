# Lecture 3 : Basic reconstruction using Python

Title : Basic reconstruction using Python

Schedule : June 11, 2020 | 16:00-17:00 

Speaker: Valéry Ozenne

## Summary

 - [Foreword](#ferword)
 - [Installation](#installation)
 - [Sequence and Data](#sequence-and-data)
 - [Objectives](#objectives)
 - [A typical Python Gadget](#a-typical-python-gadget)
 - [My first Python Gadget](#my-first-python-gadget)
   - [Folder structure](#folder-structure)
   - [Writing the gadget](#writing-the-gadget)
   - [Compilation and installation](#compilation-and-installation)
   - [Reconstruction and visualisation](#demandes-de-fusion)
   - [Exercice 1 : find the number of readout](#exercice-1--find-the-number-of-readout)
   - [Exercice 2 : display the matrix size](#demandes-de-fusion)
   - [Exercice 3 : display the AcquisitionHeader](#demandes-de-fusion)
   - [Exercice 4 : readout selection and corresponding encoding space](#demandes-de-fusion)
   - [Exercice 5 : Buffering](#buffering)
   - [Exercice 6 : Matplotlib and Ghost Niquist correction](#matplotlib-and-ghost niquist)
   - [Exercice 6 : Fourier Transform](#exercice-6-:-fourier-transform)
   - [First conclusion](#first-conclusion)
 - [A brief description of the class used to store readout, kspace or image data](#)
    - [Readout](#les-issues)
    - [Kspace](#faq)
    - [Image](#liens)
 - [My First Data Buffered Gadget](#glossaire)
    - [Writing the Buffered Gadget](#les-issues)
    - [Compilation](#faq)
    - [Exercice 1: Fourier Transform using imsmrmrd-python-tool](#liens)
    - [Exercice 2: Fourier Transform using Sigpy](#liens)
    - [Exercice 3: Fourier Transform using BART](#liens)
    - [Exercice 4: Grappa reconstruction using PyGrappa](#liens)


## Foreword 

The Gadgetron responds to two major issues in MRI:
- prototyping: how to develop a new reconstruction and associate it with an existing or developing sequence.
- deployment: how to deploy a sequence and reconstruction on several sites for a clinical study

The Gadgetron also offers software flexibility (choice of language used) and hardware flexibility (choice of reconstruction hardware: from simple PC to Cloud). Specific aera of research impose constraints on reconstruction time or latency. Typically deployment on clinical sites or interventional imaging are two scenarios where the use of C ++ will be preferable. For most of the other thematics, languages ​​such as Python or Matlab are more accessible and particularly adapted to our computing problems (ex: matrix calculation, linear algebra). Addionnally, Python is quite popular in image processing (itk, vtk) and in machine learing (keras, tensor flow). 

## Installation

To do the tutorial, you need to install two components:

* gadgetron
* python-gadgetron

Detailed installation instructions have been summarized [here](https://github.com/gadgetron/GadgetronOnlineClass/tree/master/Installation). But basically, on Ubuntu you need to run the following line:

```
sudo add-apt-repository ppa:gradient-software/experimental
sudo apt-get update
sudo apt-get install gadgetron-all
sudo pip3 install gadgetron
```

Optional

The following programm will be used at the end of the tutorial. You can skip this part at the beginning.

[ismrmrd-python-tools](https://github.com/ismrmrd/ismrmrd-python-tools)
[sigpy](https://github.com/mikgroup/sigpy-mri-tutorial)
[pygrappa](https://github.com/mckib2/pygrappa)
[ismrmrdviewer](https://github.com/ismrmrd/ismrmrdviewer)
[ismrmrd-viewer](https://github.com/DietrichBE/ismrmrd-viewer)  (an alternative)
[BART](https://github.com/mrirecon/bart)

You can install them using **pip3 install** or using the command **python3 setup.py install** after downloading the source. Nevertheless somes dependencies must be satisfied.

```
git clone https://github.com/ismrmrd/ismrmrd-python-tools.git
cd ismrmrd-python-tools
sudo python3 setup.py install
```   

```
sudo pip3 install pygrappa
sudo pip3 install sigpy
```


## Sequence and Data

We will use single-shot gradient-echo EPI acquisitions from the [CMRR sequence](https://www.cmrr.umn.edu/multiband/) acquired on a 3T Prisma from Siemens.

Data is available at this link [![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.3777994.svg)](https://doi.org/10.5281/zenodo.3777994):

Three datasets (including noise calibration and kspace) are available: 
- phantom, 12 slices, 3 repetitions, in-plane acceleration none, slice acceleration none   
- phantom, 12 slices, 3 repetitions, in-plane acceleration 2, slice acceleration none  
- brain, 36 slices, 3 repetitions, in-plane acceleration 2, slice acceleration 2 

The data has been converted with **siemens_to_ismrmrd**, we will not discuss data conversion here. This will be the object of the following readings.

## Objectives

- to become familiar with the Cartesian reconstruction pipeline
- to create new python gadget from scratch
- to create a new xml configuration file 
- data manipulation (readout, kspace, image)
- to call BART from a python gagdet
- to call SigPy from a python gagdet

## A typical Python Gadget


```python
import numpy as np
import gadgetron
import ismrmrd

def EmptyPythonGadget(connection):
   
   for acquisition in connection:
          
       # DO SOMETHING     
       connection.send(acquisition)
  

```
The function is responsible for receiving all messages from the previous gadget and for sending a new message to the next gadget using **connection.send()**
It may or may not interact with the information contained in the message.  

## My first Python Gadget

Create a new directory named `GT_Lecture3` and open two terminals at the location.

```
mkdir GT_Lecture3
cd GT_Lecture3
```

### Writing the gadget

Create the file my_first_python_gadget.py then copy the previous class. 

```
We can add the following message before connection.send() that we are going through it.
print("so far, so good")
```


### Writing the XML chain 

We will now create a new xml file named `external_python_tutorial.xml`. Add the following content into `external_python_tutorial.xml`

```xml
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

To call our gadget, we have to add it to the reconstruction chain which is currently empty. For this add the following lines after the `` MRIImageWriter``

```xml
<stream>  
        <external>
            <execute name="my_first_python_gadget" target="EmptyPythonGadget" type="python"/>
            <configuration/>
        </external> 
</stream>
```

### Compilation and installation 

Nothing to do.

### Reconstruction and visualisation

To run the reconstruction chain, you'll need to run Gadgetron, and the Gadgetron ISMRMRD client in two different terminal located in the same folder.

Start Gadgetron:
```bash
$ gadgetron
```

Run the ISMRMRD client: 
```bash 
$ gadgetron_ismrmrd_client -f Data/meas_MID00026_FID49092_cmrr_12s_80p_MB0_GP0.h5  -C external_python_tutorial.xml
```

You will see from the Gadgetron ISMRMRD client side :

``` bash
Gadgetron ISMRMRD client
  -- host            :      localhost
  -- port            :      9002
  -- hdf5 file  in   :      /home/valery/DICOM/2017-09-14_IBIO/meas_MID00026_FID49092_cmrr_12s_80p_MB0_GP0.h5
  -- hdf5 group in   :      /dataset
  -- conf            :      default.xml
  -- loop            :      1
  -- hdf5 file out   :      out.h5
  -- hdf5 group out  :      2020-05-19 15:44:55
This measurement has dependent measurements
  Noise : 16
Querying the Gadgetron instance for the dependent measurement: 66056_63650129_63650134_16
```


You will see from the Gadgetron server side :

``` bash
gadgetron
05-19 15:44:53.650 INFO [main.cpp:49] Gadgetron 4.1.1 [0c29a9433eb8dd93426ddc318f249d38af7b3d39]
05-19 15:44:53.650 INFO [main.cpp:50] Running on port 9002
05-19 15:44:53.650 INFO [Server.cpp:25] Gadgetron home directory: "/usr/local"
05-19 15:44:53.650 INFO [Server.cpp:26] Gadgetron working directory: "/tmp/gadgetron/"
05-19 15:44:55.499 INFO [Server.cpp:42] Accepted connection from: ::ffff:127.0.0.1
05-19 15:44:55.499 INFO [ConfigConnection.cpp:113] Connection state: [CONFIG]
05-19 15:44:55.501 INFO [HeaderConnection.cpp:82] Connection state: [HEADER]
05-19 15:44:55.501 INFO [VoidConnection.cpp:38] Connection state: [VOID]
05-19 15:44:55.502 DEBUG [Stream.cpp:52] Loading Gadget NoiseSummary of class NoiseSummaryGadget from 
05-19 15:44:55.537 DEBUG [NoiseSummaryGadget.cpp:35] Noise dependency folder is /tmp/gadgetron/
05-19 15:44:55.538 DEBUG [Gadget.h:130] Shutting down Gadget ()
05-19 15:44:55.539 INFO [Core.cpp:76] Connection state: [FINISHED]
05-19 15:44:55.540 INFO [Server.cpp:42] Accepted connection from: ::ffff:127.0.0.1
05-19 15:44:55.544 INFO [ConfigConnection.cpp:113] Connection state: [CONFIG]
05-19 15:44:55.558 INFO [HeaderConnection.cpp:82] Connection state: [HEADER]
05-19 15:44:55.574 INFO [StreamConnection.cpp:75] Connection state: [STREAM]
05-19 15:44:55.575 DEBUG [Stream.cpp:64] Loading External Execute block with name my_first_python_gadget of type python 
05-19 15:44:55.597 INFO [External.cpp:69] Waiting for external module 'my_first_python_gadget' on port: 46825
05-19 15:44:55.603 INFO [Python.cpp:31] Started external Python module (pid: 11056).
/usr/lib/python3/dist-packages/h5py/__init__.py:36: FutureWarning: Conversion of the second argument of issubdtype from `float` to `np.floating` is deprecated. In future, it will be treated as `np.float64 == np.dtype(float).type`.
  from ._conv import register_converters as _register_converters
05-19 15:44:55.887 DEBUG [ext. 11056 my_first_python_gadget.EmptyPythonGadget] Starting external Python module 'my_first_python_gadget' in state: [ACTIVE]
05-19 15:44:55.887 DEBUG [ext. 11056 my_first_python_gadget.EmptyPythonGadget] Connecting to parent on port 46825
05-19 15:44:55.889 INFO [External.cpp:86] Connected to external module 'my_first_python_gadget' on port: 46825
so far, so good
so far, so good
[...]
so far, so good
so far, so good
05-19 15:44:58.966 DEBUG [ext. 11056 my_first_python_gadget.EmptyPythonGadget] Connection closed normally.
05-19 15:44:59.011 INFO [Core.cpp:76] Connection state: [FINISHED]

```


### Exercice 1 : find the number of readout

Add the following lines, the initialisation must be before the loop

```python
counter=0
counter=counter+1;
print(counter)
```

Save the python file and launch the client.

### Exercice 2 : display the matrix size

Add the following lines

```
import numpy as np
print(np.shape(acquisition.data))
```

### Exercice 3 : display the AcquisitionHeader

Add the following lines

```python
print(acquisition.active_channels)
print(acquisition.scan_counter)
```


### Exercice 4 : readout selection and corresponding encoding space

Get the repetion number and slice number using the following lines:

```python
slice = acquisition.idx.slice
repetition=  acquisition.idx.repetition
e1=acquisition.idx.kspace_encode_step_1
segment=acquisition.idx.segment
print(counter, " slice: ",slice , " rep: ", repetition, " e1: ", e1," segment: ",  segment)
```

Now we can add a filter using the following command before the loop:

```python
# We're only interested in repetition ==0  in this example, so we filter the connection. Anything filtered out in
# this way will pass back to Gadgetron unchanged.
connection.filter(lambda acq: acq.idx.repetition ==0)
```

Increase the selection

```python
connection.filter(lambda acq: acq.idx.repetition ==2 and acq.idx.slice ==0)
```

Increase the selection
```python 
connection.filter(lambda acq: acq.idx.repetition ==2 and acq.idx.slice ==0 and acq.is_flag_set(ismrmrd.ACQ_IS_REVERSE))
```

Increase the selection
```python 
connection.filter(lambda acq: acq.idx.repetition ==2 and acq.idx.slice ==0 and acq.is_flag_set(ismrmrd.ACQ_IS_REVERSE) and acq.is_flag_set(ismrmrd.ACQ_IS_PARALLEL_CALIBRATION))
```

Pick then the dataset with ACS calibration and try again


### Exercice 5 : Matplotlib & Ghost Niquist

Copy and paste the previous function and call it EpiPythonGadget.

Usign the connection filter, filter the data with the FLAGS:ACQ_IS_PHASECORR_DATA 
Pick only the first slice, you will see 9 lines using the fully sampled dataset:

```python
 counter:  1  scan_counter:  2  slice:  0  rep:  0  e1:  32  segment:  1
 counter:  2  scan_counter:  3  slice:  0  rep:  0  e1:  32  segment:  0
 counter:  3  scan_counter:  4  slice:  0  rep:  0  e1:  32  segment:  1
 counter:  4  scan_counter:  1190  slice:  0  rep:  1  e1:  32  segment:  1
 counter:  5  scan_counter:  1191  slice:  0  rep:  1  e1:  32  segment:  0
 counter:  6  scan_counter:  1192  slice:  0  rep:  1  e1:  32  segment:  1
 counter:  7  scan_counter:  2378  slice:  0  rep:  2  e1:  32  segment:  1
 counter:  8  scan_counter:  2379  slice:  0  rep:  2  e1:  32  segment:  0
 counter:  9  scan_counter:  2380  slice:  0  rep:  2  e1:  32  segment:  1
```

Now, we would like to compare the magnitude and phase of the kspace using matplotlib.

First set the import

```python
#import matplotlib
#matplotlib.use('Qt4Agg')  
import matplotlib.pyplot as plt
```

Then, pick the first channel.

```
fid=np.abs(np.squeeze(acquisition.data[0,:]))
```

and plot the data. The reverse line are plot in red and normal in blue 

```python
if (acquisition.is_flag_set(ismrmrd.ACQ_IS_REVERSE)):
          plt.plot(fid, 'r')
       else:
          plt.plot(fid, 'b') 
       if (counter%3==0):  
          plt.show()
          plt.pause(2)
```

Note that when you close the matplotlib window, the reco continue to the next plot.

The lines are used to correct the Ghost-Niquist artefact (caused by gradient imperfection) by computing the phase difference between positive and negative readout after ifft in the readout direction. Such corrections has been implemented in C++ as well as the regridding. Please see Generic_Cartesian_Grappa_EPI.xml.

### Exercice 6 : Buffering

For the last exercice : copy and paste my_first_python_gadget.py into my_second_python_gadget.py

In order to do the buffering, we need to first find the dimension of the ksapce and to allocate the matrix.
The flexible header or **ISMRMRDHeader** include general information about the acquisition like **patientInformation**, **acquisitionSystemInformation** and **encoding** information.
The ISMRMRD format will be explained in the next lectures.


```python
h=connection.header

number_of_channels=h.acquisitionSystemInformation.receiverChannels   
   
encoding_space = h.encoding[0].encodedSpace
              
eNz = encoding_space.matrixSize.z
eNy = encoding_space.matrixSize.y
eNx = encoding_space.matrixSize.x

encoding_limits = h.encoding[0].encodingLimits

number_of_slices=encoding_limits.slice.maximum+1
number_of_repetitions=encoding_limits.repetition.maximum+1

print("[RO, E1, E2, CHA, SLC, REP ]: ", eNx, eNy  , eNz , number_of_channels, number_of_slices, number_of_repetitions)
```

Then we do the allocation usign numpy

```python
mybuffer=np.zeros(( int(eNx),int(eNy), int(eNz), int(number_of_channels)),dtype=np.complex64)
```

Then in the loop, get the encoding index and compare the size of the buffer and the readout:

```
e1=acquisition.idx.kspace_encode_step_1
e2=acquisition.idx.kspace_encode_step_2
slice = acquisition.idx.slice
repetition=  acquisition.idx.repetition   

print(np.shape(acquisition.data))
print(np.shape(mybuffer[:,e1,e2,:])) 
```

There is an oversampling in readout direction by a factor of 2 in all Siemens acquisition.
Use `np.transpose` to copy and paste the data


### First Conclusion

This conclude the lecure on readout. Note that basic kspace processing step have already been developped in python or in C++.
There is no need to redoo it except for educational purpose. Call them in xml.


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

In cartesian sampling, two gadgets play a fundamental role : AcquisitionAccumulateTriggerGadget and BucketToBufferGadget. 

These gadgets are used to buffer readouts in order to build the kspace. In MRI, the dimensions are very numerous:: 

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
 
By convention, in input the matrix size is [RO, CHA] and in output is [RO E1 E2 CHA N S SLC].
The dimensions **N** and **S** are chosen by the user. 

It is very interesting to position yourself after these gadgets where the kspaces data are automatically sorted, whether it is the calibration lines in parallel imaging or the lines sampled.   


The calibration data if present is accessible via the following structure:

```
buffer.ref.data  
buffer.ref.header
```

The fullysampled or undersampled data are accessible via the following structure:

```
buffer.data.data
buffer.data.header
```

Be cautious, the size of the headers is associated with the size of the data. Between them the headers are generally different, for example the position of the slicess change according to the SLC direction. We now have a hoNDarray acquisitionHeader with a matrix size of [E1 E2 N S SLC]. The headers being identical according to the direction of readout and for all channels.

 
### Image


### Writing the Buffered Gadget

Nous allons donc maintenant créer un nouveau gadget nommé `SimpleDataBufferedPythonGadget` dans un fichier appelé `my_first_buffered_data_gadget.py`

`

```
import numpy as np
import gadgetron
import ismrmrd
import logging
import time

def SimplePythonGadget(connection):
   logging.info("Python reconstruction running - reading readout data")
   start = time.time()
   counter=0

   for acquisition in connection:          
        
      
       connection.send(acquisition)

   logging.info(f"Python reconstruction done. Duration: {(time.time() - start):.2f} s")
```

And a new xml file named `external_python_buffer_tutorial.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <version>2</version>

    <readers>
        <reader>
            <dll>gadgetron_core_readers</dll>
            <classname>AcquisitionReader</classname>
        </reader>
        <reader>
            <dll>gadgetron_core_readers</dll>
            <classname>WaveformReader</classname>
        </reader>
    </readers>

    <writers>
        <writer>
            <dll>gadgetron_core_writers</dll>
            <classname>ImageWriter</classname>
        </writer>
    </writers>

    <stream> 

       <gadget>
            <dll>gadgetron_mricore</dll>
            <classname>NoiseAdjustGadget</classname>
        </gadget>
        
          <!-- EPI correction -->
     <gadget>
        <name>ReconX</name>
        <dll>gadgetron_epi</dll>
        <classname>EPIReconXGadget</classname>
     </gadget>

     <gadget>
        <name>EPICorr</name>
        <dll>gadgetron_epi</dll>
        <classname>EPICorrGadget</classname>
     </gadget>

     <gadget>
        <name>FFTX</name>
        <dll>gadgetron_epi</dll>
        <classname>FFTXGadget</classname>
     </gadget>

     <gadget>
        <name>OneEncodingSpace</name>
        <dll>gadgetron_epi</dll>
        <classname>OneEncodingGadget</classname>
      </gadget>


       <!-- Data accumulation and trigger gadget -->
    <gadget>
        <name>AccTrig</name>
        <dll>gadgetron_mricore</dll>
        <classname>AcquisitionAccumulateTriggerGadget</classname>
        <property><name>trigger_dimension</name><value>repetition</value></property>
        <property><name>sorting_dimension</name><value></value></property>
    </gadget>

      <gadget>
        <name>BucketToBuffer</name>
        <dll>gadgetron_mricore</dll>
        <classname>BucketToBufferGadget</classname>
        <property><name>N_dimension</name><value>contrast</value></property>
        <property><name>S_dimension</name><value>average</value></property>
        <property><name>split_slices</name><value>false</value></property>
        <property><name>ignore_segment</name><value>true</value></property>
     </gadget>

        <external>
            <execute name="my_first_data_buffered_python_gadget" target="SimpleDataBufferedPythonGadget" type="python"/>
            <configuration/>
        </external>
 
    </stream>

</configuration>
```

### Compilation 

Nothing to do.

### Reconstruction and visualisation

To run the reconstruction chain, you'll need to run Gadgetron, and the Gadgetron ISMRMRD client in two different terminal located in the same folder.

Start Gadgetron:
```bash
$ gadgetron
```

Run the ISMRMRD client: 
```bash 
$ gadgetron_ismrmrd_client -f Data/meas_MID00026_FID49092_cmrr_12s_80p_MB0_GP0.h5  -C external_python_tutorial.xml
```

You will see from the Gadgetron ISMRMRD client side :

``` bash
Gadgetron ISMRMRD client
  -- host            :      localhost
  -- port            :      9002
  -- hdf5 file  in   :      /home/valery/DICOM/2017-09-14_IBIO/meas_MID00026_FID49092_cmrr_12s_80p_MB0_GP0.h5
  -- hdf5 group in   :      /dataset
  -- conf            :      default.xml
  -- loop            :      1
  -- hdf5 file out   :      out.h5
  -- hdf5 group out  :      2020-05-19 15:44:55
This measurement has dependent measurements
  Noise : 16
Querying the Gadgetron instance for the dependent measurement: 66056_63650129_63650134_16
```


You will see from the Gadgetron server side :

```bash
[...]

```

### Exercice 1: Fourier Transform using imsmrmrd-python-tool


```python
import matplotlib.pyplot as plt
from ismrmrdtools import show, transform
```

```python
for acquisition in connection:
      
       #acquisition is a vector of a specific structure, we called reconBit 
       #print(type(acquisition[0]))

       for reconBit in acquisition:

           print(type(reconBit))
           # reconBit.ref is the calibration for parallel imaging
           # reconBit.data is the undersampled dataset
           print('-----------------------')
	   # each of them include a specific header and the kspace data
           print(type(reconBit.data.headers))
           print(type(reconBit.data.data))

           print(reconBit.data.headers.shape)
           print(reconBit.data.data.shape)
           
           repetition=reconBit.data.headers.flat[34].idx.repetition 
           print(repetition)
```

we could set alternative names for acquisition and reconBit but then data, ref and data and headers are fixed and refered to a specific class.

```python
for lala in connection
  for lili in lala
      #use
      lili.ref.headers
      lili.ref.data
      lili.data.headers
      lili.data.data
      
```

### Exercice 2: Fourier Transform using BART
### Exercice 3: Fourier Transform using Sigpy
### Exercice 4: Grappa reconstruction using PyGrappa





