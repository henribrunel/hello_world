#!/usr/bin/env python

"""
Describes the class: Library_Spectroscopy


"""

import numpy as np
import xlrd
from matplotlib import cm     
import PYRATS.config as config
from PYRATS.Materials import Gas

import os


class Library_Spectroscopy():
    '''
    This class build a library object for spectroscopy, and extends the dictionnary class.
    
    Keys of the dictionnary are those of the database. For example, C02 is identified as '02' in HITRAN2012. The values of 
    the dictinnary are the location of the text file of the database
    


    '''
    
    
    def __init__(self,):
        
        super(Library_Spectroscopy,self).__init__()#Building the dictionnary object
        
        '''
        set the functions of the class
        - associate a gas to the library
        - associate a range of number of wave
        '''
        
    def set_gas_library(self,Gas):
    
        self.Gas=Gas
    
        return
    
    def get_gas_library(self,):
        
        return self.Gas
        
        
    def set_range_freq(self, vinf, vsup):
        
        self.freq_range=np.array([vinf,vsup],dtype=float)
        
        return
        
    def get_range_freq(self,):
        
        return self.freq_range
        
        
    def get_informations(self, ):
        
        '''
        return an array with the Hitran values of the gas
        for each number of wave in the freq range defined, you get the 10 Hitran values
        '''
        
        try:
            self.get_gas_library()
        except AttributeError:
            print('no gas is attributed to the library')
            return
        try:
            self.get_range_freq()
        except AttributeError:
            print('range of number of wave undefined')
            return
            
        try:
            dic_locations=config.dic_location_Hitran
            fileID=open(dic_locations[self.Gas.IDMolecule],'r').readlines()
        except KeyError:
            print('gas not in HITRAN')
            return
        
        
        # Key Dictionnary for seeking the values of the Gas parameters
        CleHit = {
        'M'          : {'pos' :   1,   'len' :  2,   'format' : '%2d' },
        'I'          : {'pos' :   3,   'len' :  1,   'format' : '%1d' },
        'nu'         : {'pos' :   4,   'len' : 12,   'format' : '%12f'},
        'S'          : {'pos' :  16,   'len' : 10,   'format' : '%10f'},
        'R'          : {'pos' :  26,   'len' :  0,   'format' : '%0f' },
        'A'          : {'pos' :  26,   'len' : 10,   'format' : '%10f'},
        'gamma_air'  : {'pos' :  36,   'len' :  5,   'format' : '%5f' },
        'gamma_self' : {'pos' :  41,   'len' :  5,   'format' : '%5f' },
        'E_'         : {'pos' :  46,   'len' : 10,   'format' : '%10f'},
        'n_air'      : {'pos' :  56,   'len' :  4,   'format' : '%4f' },
        'delta_air'  : {'pos' :  60,   'len' :  8,   'format' : '%8f' },
        'g'          : {'pos' : 147,   'len' :  7,   'format' : '%7f' },
        'g_'         : {'pos' : 154,   'len' :  7,   'format' : '%7f' }
        }
             
        #fix the range
        
        vinf=self.freq_range[0]
        vsup=self.freq_range[1]
        
        
        '''
        #Compute the size of the Hitran array 

        '''
        
        '''
        Algorithme of Dichotomia
        '''
        p0=0
        p1=len(fileID)

        while (p1-p0)>1:
            m=int((p1+p0)/2)
            line0=fileID[m]
            freq_name=line0[CleHit['nu']['pos']:(CleHit['nu']['pos']+CleHit['nu']['len'])]
            freq_number=float(freq_name)
            if freq_number>vinf:
                p1=m
                p0=p0
            else:
                p1=p1
                p0=m
                
        #Store the values into the array
        listofparam=['nu','S','A','gamma_air','gamma_self','E_','n_air','delta_air','g','g_']        

        nl=0
        while freq_number<vsup:
            line=fileID[p1]
            line=' '+line
            iso_name=line[CleHit['I']['pos']:(CleHit['I']['pos']+CleHit['I']['len'])]
            iso_number=int(iso_name)
            freq_name=line[CleHit['nu']['pos']:(CleHit['nu']['pos']+CleHit['nu']['len'])]
            freq_number=float(freq_name)
            iso_number_int=int(self.Gas.IDIsotope)
            p1=p1+1
            if  iso_number==iso_number_int:
                nl=nl+1
                
        #Set the initial values of the parameters
        value_array=np.zeros((nl,10),dtype=float)
        nl=-1
        p1=m+1
        line=fileID[p1]
        line=' '+line
        freq_name=line[CleHit['nu']['pos']:(CleHit['nu']['pos']+CleHit['nu']['len'])]
        freq_number=float(freq_name)
        
        #run the loop
        while freq_number<vsup:
            line=fileID[p1]
            line=' '+line
            iso_name=line[CleHit['I']['pos']:(CleHit['I']['pos']+CleHit['I']['len'])]
            iso_number=int(iso_name)
            freq_name=line[CleHit['nu']['pos']:(CleHit['nu']['pos']+CleHit['nu']['len'])]
            freq_number=float(freq_name)
            iso_number_int=int(self.Gas.IDIsotope)
            p1=p1+1
            if  iso_number==iso_number_int:
                nl=nl+1
                for j in range(len(listofparam)):
                    k=listofparam[j]
                    valuename=line[CleHit[k]['pos']:(CleHit[k]['pos']+CleHit[k]['len'])]
                    valuenumber=float(valuename)
                    value_array[nl,j]=valuenumber
                    
        return value_array
