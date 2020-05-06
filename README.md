# Tuto_GT
Tutorial for OnlineCourse

# Données et acquisitions

Nous allons utiliser des acquisitions provenant de la séquence SMS du CMRR: https://www.cmrr.umn.edu/multiband/ acquises sur une Prisma Siemens 3T.

Les données sont disponibles à ce [lien](https://zenodo.org/), elles contient:

- acquisition single-shot grandient EPI, 12 slices, 3 repetitions, in-plane acceleration none, slice acceleration none   
- acquisition single-shot grandient EPI, 12 slices, 3 repetitions, in-plane acceleration 2, slice acceleration none  
- acquisition single-shot grandient EPI, 12 slices, 3 repetitions, in-plane acceleration 2, slice acceleration 2 

Les données ont été converties avec siemens_to_ismrmrd, nous n'aborderons pas ici la conversion des données. Ce sera l'object des lectures suivantes. 

## Objectifs de ce tutorial

- se familiariser avec le pipeline de reconstruction cartesienne
- créer des nouveaux gadgets python
- manipuler les données kspace et image
- appeler bart depuis un gadget python
- appeler sigpy depuis un gadget python


## 1) Le 

 
