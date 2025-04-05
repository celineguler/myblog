---
layout: post
title: "Elec. Struct. for QED Cavity Mols."
---

  <!-- MathJax Script -->
  <script type="text/javascript" async
    src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js">
  </script>

 
{% highlight ruby%}



import numpy as np
from pathlib import Path 
import sys
import re
import matplotlib.pyplot as plt

# Reading the experimental data from script
def is_valid(line):
    return bool(re.match(r'^[\d,\s]+$', line.strip()))

def read_file(file_path):

    path = Path(file_path)

    if not path.exists():
        print(f"Error: File '{file_path}' does not exist.")
        sys.exit(1)

    with path.open("r", encoding="utf-8", errors="ignore") as file:
        content = [line.strip().replace(',', '.') for
        line in file.readlines() if is_valid(line)]

    return content

# data_split splits unnecessary characters from script to access numerical data
def data_split(content):
    data = []
    for line in content:
        values = line.split('\t')
        data.append([float(values[0]), float(values[1]), float(values[2])])
    return data


def main():
    
    # sys.argv[1] takes w/absorber and sys.argv[2] takes wo/absorber data.
    data = sys.argv[1]
    absorber = read_file(data)
    data1 = data_split(absorber)
    print("Data1:", data1)
    return data1
       
if __name__ == "__main__":
    main()


# plot the data.

data = main()
x_values = [row[0] for row in data]
y_values = [row[1] for row in data]

plt.figure(figsize=(8, 5))
plt.plot(x_values, y_values, linestyle='--', label="Franck-Hertz Experiment")
plt.xlabel("U")
plt.ylabel("I")
plt.title("I-U Curve of Franck-Hertz Experiment")
plt.grid(True)
plt.legend()
plt.savefig("plot.png")


plt.figure(figsize=(8, 5))
plt.plot(x_values, y_values, linestyle='--', label="Franck-Hertz Experiment")
plt.xlabel("U")
plt.ylabel("I")
plt.title("I-U Curve of Franck-Hertz Experiment")
plt.grid(True)
plt.legend()
plt.xlim(20, 46)
plt.savefig("plot_2.png")




{% endhighlight %}






I think it would be interesting if we put COF-1 inside an optical cavity and try to construct the Elec. Struct.







{% highlight ruby%}

'''
GReExp optimize BiTeI unit cell M3GNet+D3 FrechetCellFilter BFGSLineSearch conda:py310x
'''
import os
import sys

from scipy.constants import angstrom      ## one Angstrom in meters
from scipy.constants import atmosphere    ## standard atmosphere in pascals
from scipy.constants import Boltzmann     ## Boltzmann constant
from scipy.constants import electron_mass ## electron mass
from scipy.constants import electron_volt ## one electron volt in Joules
from scipy.constants import giga
from scipy.constants import hbar          ## h/2pi
from scipy.constants import nano
from scipy.constants import milli
from scipy.constants import physical_constants
from scipy.constants import Planck        ## the Planck constant h
from scipy.optimize import curve_fit

import sympy as sp

from ase import Atom, Atoms, build
from ase.build import stack, surface
from ase.cell import Cell
from ase.constraints import FixAtoms, FixSymmetry
from ase.dft.kpoints import BandPath
from ase.filters import FrechetCellFilter
from ase.formula import Formula
from ase.geometry import get_distances
from ase.io import read, write
from ase.io.trajectory import Trajectory
from ase.lattice import HEX
from ase.lattice.cubic import FaceCenteredCubic
from ase.optimize import FIRE
from ase.optimize.bfgslinesearch import BFGSLineSearch
from ase.spacegroup.symmetrize import check_symmetry, refine_symmetry
from ase.units import GPa
from ase.build import bulk
from ase.calculators.emt import EMT
from ase.eos import calculate_eos
from ase.units import kJ
from ase.visualize import view
import datetime
from importlib.metadata import version, PackageNotFoundError
import importlib.util
from io import StringIO
from ase.eos import EquationOfState
import numpy as np
from scipy.interpolate import bisplev, InterpolatedUnivariateSpline, RectBivariateSpline
from scipy.optimize import minimize, minimize_scalar
import math
import matplotlib.pyplot as plt
from matplotlib import rc, rcParams
import matplotlib.gridspec as gridspec

rc('text', usetex=True)
plt.rcParams['text.latex.preamble'] = r"\usepackage{amsmath}"

import warnings
import logging
logging.basicConfig(filename='warnings.log', level=logging.WARNING)
logger = logging.getLogger()
def warning_to_log(message, category, filename, lineno, file=None, line=None):
  logger.warning(f'{filename}:{lineno}: {category.__name__}: {message}')
warnings.showwarning = warning_to_log

print("-------------------------------------")
print(f"## Save version info as of {datetime.datetime.now():%Y-%m-%d}")
packages = ["dgl", "pymatgen", "ase", "numpy", "matgl", "dftd3"]
for pkg in packages:
  try:
    pkg_version = version(pkg)
    pkg_location = importlib.util.find_spec(pkg).submodule_search_locations[0]
  except PackageNotFoundError:
    pkg_version = "Not installed"
    pkg_location = "N/A"
  except AttributeError:
    pkg_location = "Location not found"
  print(f"## {pkg:<11} {pkg_version:<10} {pkg_location}")
print("-------------------------------------")



print("------------------------")
print("## Calculator: M3GNet")
from m3gnetd3calculator import m3gnet, m3gnetd3, m3gnetd4
calc = m3gnet()
calcd3 = m3gnetd3()
calcd4 = m3gnetd4()
print("------------------------")



print("------------------------------------")
print("## Output directory <-- sys.argv[1]:",end=' ')
outdir=sys.argv[1]
if not os.path.exists(outdir):
  os.makedirs(outdir)
print(outdir)
print("------------------------------------")


print("----------------------------")
print("## E vs V using 3D unit cell")
if not os.path.exists(outdir):
  os.makedirs(outdir)
  print("## Directory made:",outdir)
pdffil=os.path.join(outdir,'EvsV.pdf')


volumes = np.arange(10.5,20.7,0.5)
volumes = np.append(volumes,19.190)
volumes = np.sort(volumes)
energies = []
energiesd3 = []
energiesd4 = []
for vol in volumes:
  tag = f"vol{vol:07.2f}".replace('.', '')
  ciffil=os.path.join(outdir,tag+'.cif')
  print(f"Checking file: {ciffil}")
  datfil=ciffil.replace('.cif','.dat')
  if not os.path.lexists(datfil):
    if os.path.lexists(ciffil):
      stru=read(ciffil)
      print(f'## File read:',ciffil)
    else:
      print(f'## File absent:',ciffil)
      stru = bulk('MgO', crystalstructure='rocksalt', a=np.power(4*vol, 1/3.0))
    stru.calc = calc
    E = stru.get_potential_energy()
    energies.append(E)
    stru.calc = calcd3
    Ed3 = stru.get_potential_energy()
    energiesd3.append(Ed3)
    stru.calc = calcd4
    Ed4 = stru.get_potential_energy()
    energiesd4.append(Ed4)
    write(ciffil, stru)
    print("## File written:", ciffil)
    symm = check_symmetry(stru, symprec=0.0001, verbose=False)
    fu = stru.get_chemical_formula(empirical=True)
    Z = Formula(str(stru.symbols)).stoichiometry()[2]
    maxforce = np.max(np.abs(stru.get_forces()))
    stress = stru.get_stress() / GPa
    V = stru.get_volume()
    N = len(stru)
    with open(datfil, 'w') as f:
      f.write(f"{V:8.3f} {E:13.6f} ")
      f.write(f"{fu:12s} {Z:2d} {N:5d} ")
      f.write(f"{maxforce:8.4f} ")
      f.write(f"{stress[0]:8.3f} {stress[1]:8.3f} {stress[2]:8.3f} {stress[3]:8.3f} {stress[4]:8.3f} {stress[5]:8.3f} ")
      f.write(f"{symm['number']:3d} ")
      f.write(f"## V(A^3) E(eV) fu Z N maxforce(eV/A) stress(GPa) ITA\n")
    print(f'## File written:', datfil)
  else:
    with open(datfil, 'r') as f:
      lines = f.readlines()
    for line in lines:
      data = line.split()
      if 1e-6 < np.abs(vol - float(data[0])):
        print(f'## ',vol,' .NE.',data[0])
      E = float(data[1])
      energies.append(E)
    

  
  
#  supercell=build.sort(build.make_supercell(stru,np.array([[-1,1,1],[1,-1,1],[1,1,-1]])))
#  supercell.calc = calc
#  Ed3 = supercell.get_potential_energy()
#  write("supercell.cif", supercell)
traj = Trajectory('MgO.traj', 'w')


spline2 = InterpolatedUnivariateSpline(volumes, energies, k=3)
result = minimize_scalar(spline2, bounds=(volumes.min(), volumes.max()), method='bounded')
volume_min = result.x
energy_min = spline2(volume_min)

Vfine = np.linspace(volumes.min(), volumes.max(), 500)
Einterpolated = spline2(Vfine)
cifunicelopt=os.path.join(outdir,'volopt.cif')
datfil=cifunicelopt.replace('.cif','.dat')





# PRESSURE VS VOLUME

eos = EquationOfState(volumes, energies, eos='birchmurnaghan')
v0, e0, B = eos.fit()
eos.plot(show=True)

eos2 = EquationOfState(volumes, energiesd3, eos='birchmurnaghan')
v0, e0, B = eos2.fit()

eos3 = EquationOfState(volumes, energiesd4, eos='birchmurnaghan')
v0, e0, B = eos3.fit()

bulk_modulus = eos.B / kJ * 1.0e24
bulk_at_zero = eos.eos_parameters[2]

bulk_modulus2 = eos2.B / kJ * 1.0e24
bulk_at_zero2 = eos2.eos_parameters[2]

bulk_modulus3 = eos3.B / kJ * 1.0e24
bulk_at_zero3 = eos3.eos_parameters[2]


# birch murnagham pressure
def pressure(V, V0, K0, K0_prime):
    P = (3 * K0 / 2) * (np.power(V0 / V, 7/3.0) - np.power(V0 / V, 5/3.0)) * \
        (1 + (3/4) * (K0_prime - 4) * (np.power((V0 / V), 2/3) - 1))
    return P

def pressure5th(V,V0, K0, K0_prime,K0_prime2, K0_prime3):
  f = (1/2)*(np.power(V0 / V, 2/3) - 1)
  a1 = (3/2)*(K0_prime-4)
  a2 = (3/2)*(K0*K0_prime2 + K0_prime*(K0_prime - 7) + 143/7)
  a3 = (1/8)*(9*np.power(K0_prime,2)*K0_prime3 + 12*(3*K0_prime - 8)*K0*K0_prime2 + K0_prime*(118 + np.power(3*K0_prime-16,2)) - 1888/3)
  P=3*K0*f*np.power(1+2*f, 5/2)*(1+a1*f+a2*np.power(f,2)*a3*np.power(f,3))
  return P


pressures = []
pressuresd3=[]
pressuresd4=[]
pressuresPAW=[]
pressuresECP=[]
pressure5=[]
pressure0K=[]
pressurestat=[]

step = (eos.v0 - 10.36) / len(volumes)
volumes_plot = np.linspace(10.74, eos.v0, len(volumes))
# volumes_plot = np.arange(10.52, eos.v0, step)

for vol in Vfine:
  P = pressure(vol, eos.v0, bulk_modulus, bulk_at_zero)
  Pd3 = pressure(vol, eos2.v0, bulk_modulus2, bulk_at_zero2)
  Pd4 = pressure(vol, eos3.v0, bulk_modulus3, bulk_at_zero3)
  P_Paw_Large = pressure(vol, 76.049/4,  154.183, 4.141)
  P_Ecp_Large = pressure(vol, 77.629/4, 151.707, 4.212)
  P_5th = pressure5th(vol, 18.76,  161.5, 4.0, -0.026, 0.0013)
  #pressurestatic=pressure(vol,)
  pressures.append(P)
  pressuresd3.append(Pd3)
  pressuresd4.append(Pd4)
  pressuresPAW.append(P_Paw_Large)
  pressuresECP.append(P_Ecp_Large)
  pressure5.append(P_5th)


# 2003 oganov

def E(V, V0_oganov, E0, K0, K0_prime):
  K0 *= GPa
  x = V0_oganov / V
  xi = (3/4) * (K0_prime - 4)
    
  term1 = (3/2) * (xi - 1) * np.power(x, (2/3))
  term2 = (3/4) * (1 - 2 * xi) * np.power(x,(4/3))
  term3 = (1/2) * xi * np.power(x,(6/3))
  constant_term = (2 * xi - 3) / 4

  E_x = E0 + (3/2) * K0 * V0_oganov * (term1 + term2 + term3 - constant_term)
    
  return E_x


#energies
E_static = E(Vfine, 73.425/4, eos.e0, 181.240, 3.997)
E_ECP_large = E(Vfine, 77.629/4, eos.e0, 151.707, 4.212)
E_PAW_large = E(Vfine, 76.049/4, eos.e0, 154.183, 4.141)
E_0k = E(Vfine, 74.439/4, eos.e0, 173.480, 4.014)
Efit = E(Vfine, eos.v0, eos.e0, bulk_modulus, bulk_at_zero)
EfitD3 = E(Vfine, eos2.v0, eos2.e0, bulk_modulus2, bulk_at_zero2)
EfitD4 = E(Vfine, eos3.v0, eos3.e0, bulk_modulus3, bulk_at_zero3)


#pressures

def P_2003(K0, K0_prime, V, V0_oganov):
  Pres = - ((3/2)*K0*(xi-1)*np.power(x,-5/3)) - ((3/2)*K0*(1-2*xi)*np.power(x,-7/3)) - ((3/2)*K0*xi*np.power(x,-10/3))
  return Pres



#experimental values 
exp_pressures=[2.89, 6.73, 11.02, 15.82, 21.17, 27.15, 33.84, 41.32, 49.67, 59.02, 
             69.49, 81.22, 87.61, 94.38, 101.55, 109.15, 117.22, 125.77, 134.85, 144.49, 
             154.73, 165.60, 177.16, 189.46, 202.54, 216.47, 231.31, 247.12, 263.98, 281.97, 
             301.18, 321.70]

exp_volumes=[0.98, 0.96, 0.94, 0.92, 0.90, 0.88, 0.86, 0.84, 0.82, 0.80, 
        0.78, 0.76, 0.75, 0.74, 0.73, 0.72, 0.71, 0.70, 0.69, 0.68,
        0.67, 0.66, 0.65, 0.64, 0.63, 0.62, 0.61, 0.60, 0.59, 0.58, 
        0.57, 0.56]



fig, (axi, axi_E) = plt.subplots(1, 2, figsize=(14,8))

axi.plot(pressures, Vfine, label='T=0K', color='green',linestyle='--', linewidth=2)
#axi.plot(np.array(exp_pressures), np.array(exp_volumes)*eos.v0, label='T=300K', color='red')
axi.plot(pressuresd3, Vfine, label='T=0K M3gnet+D3', color='red',linestyle='--', linewidth=2)
axi.plot(pressuresd4, Vfine, label='T=0K M3gnet+D4', color='blue',linestyle='--', linewidth=2)
axi.plot(pressuresPAW, Vfine, label='T=0K PAW Large-core MgO', color='gray')
axi.plot(pressuresECP, Vfine, label='T=0K ECP Large-core MgO', color='purple')
axi.plot(pressure5, Vfine, label='5th order DFT-GGA', color='pink')

axi_E.plot(volumes, energies, '-o', label='M3gnet', color='green')
axi_E.plot(volumes, energiesd3, '-o', label='M3gnet+D3', color='brown')
axi_E.plot(volumes, energiesd4, '-o', label='M3gnet+D4', color='gray')
axi_E.plot(Vfine, Efit, color='green')
axi_E.plot(Vfine, EfitD3, color='brown')
axi_E.plot(Vfine, EfitD4, color='gray')
axi_E.plot(Vfine, E_ECP_large, label='ECP Large Core MgO', color='pink',linestyle='--', linewidth=2)
axi_E.plot(Vfine, E_PAW_large, label='PAW Large Core MgO', color='purple',linestyle='--', linewidth=2)
axi_E.plot(Vfine, E_static, label='2003 Oganov Static', color='red',linestyle='--', linewidth=2)
axi_E.plot(Vfine, E_0k, label='2003 Oganov 0K', color='blue',linestyle='--', linewidth=2)

axi.set_xlabel(r'$P$ (GPa)', fontsize=16)
axi.set_ylabel(r'$V(A^3)$', fontsize=16)
axi.tick_params(axis='y', labelsize=12)
axi.tick_params(axis='x', labelsize=12)
axi_E.set_ylabel(r'$E$ (eV)', fontsize=16)
axi_E.set_xlabel(r'$V(A^3)$', fontsize=16)
axi_E.tick_params(axis='y', labelsize=12)
axi_E.tick_params(axis='x', labelsize=12)
axi.legend()
axi_E.legend()
axi.set_title('Pressure vs Volume', fontsize=14)
axi_E.set_title('Energy vs Volume', fontsize=14)

plt.subplots_adjust(hspace=0.5)
plt.tight_layout()
plt.savefig(pdffil)
print('## Figure saved:',pdffil)
plt.show()
print("----------------------------")

{% endhighlight %}



{% highlight ruby%}
import os
import sys

from scipy.constants import angstrom      ## one Angstrom in meters
from scipy.constants import atmosphere    ## standard atmosphere in pascals
from scipy.constants import Boltzmann     ## Boltzmann constant
from scipy.constants import electron_mass ## electron mass
from scipy.constants import electron_volt ## one electron volt in Joules
from scipy.constants import giga
from scipy.constants import hbar          ## h/2pi
from scipy.constants import nano
from scipy.constants import milli
from scipy.constants import physical_constants
from scipy.constants import Planck        ## the Planck constant h
from scipy.optimize import curve_fit

import sympy as sp

from ase import Atom, Atoms, build
from ase.build import stack, surface
from ase.cell import Cell
from ase.constraints import FixAtoms, FixSymmetry
from ase.dft.kpoints import BandPath
from ase.filters import FrechetCellFilter
from ase.formula import Formula
from ase.geometry import get_distances
from ase.io import read, write
from ase.io.trajectory import Trajectory
from ase.lattice import HEX
from ase.lattice.cubic import FaceCenteredCubic
from ase.optimize import FIRE
from ase.optimize.bfgslinesearch import BFGSLineSearch
from ase.spacegroup.symmetrize import check_symmetry, refine_symmetry
from ase.units import GPa
from ase.units import kJ
from ase.build import bulk
from ase.calculators.emt import EMT
from ase.eos import calculate_eos
from ase.visualize import view
import datetime
from importlib.metadata import version, PackageNotFoundError
import importlib.util
from io import StringIO
from ase.eos import EquationOfState
import numpy as np
from scipy.interpolate import bisplev, InterpolatedUnivariateSpline, RectBivariateSpline
from scipy.optimize import minimize, minimize_scalar
import math
import matplotlib.pyplot as plt
from matplotlib import rc, rcParams
import matplotlib.gridspec as gridspec


rc('text', usetex=True)
plt.rcParams['text.latex.preamble'] = r"\usepackage{amsmath}"

import warnings
import logging
logging.basicConfig(filename='warnings.log', level=logging.WARNING)
logger = logging.getLogger()
def warning_to_log(message, category, filename, lineno, file=None, line=None):
  logger.warning(f'{filename}:{lineno}: {category.__name__}: {message}')
warnings.showwarning = warning_to_log

print("-------------------------------------")
print(f"## Save version info as of {datetime.datetime.now():%Y-%m-%d}")
packages = ["dgl", "pymatgen", "ase", "numpy", "matgl", "dftd3"]
for pkg in packages:
  try:
    pkg_version = version(pkg)
    pkg_location = importlib.util.find_spec(pkg).submodule_search_locations[0]
  except PackageNotFoundError:
    pkg_version = "Not installed"
    pkg_location = "N/A"
  except AttributeError:
    pkg_location = "Location not found"
  print(f"## {pkg:<11} {pkg_version:<10} {pkg_location}")
print("-------------------------------------")

print("------------------------")
print("## Calculator: M3GNet+D3")
from m3gnetd3calculator import m3gnet, m3gnetd3
calc = m3gnetd3()
print("------------------------")

print("------------------------------------")
print("## Output directory <-- sys.argv[1]:",end=' ')
outdir=sys.argv[1]
if not os.path.exists(outdir):
  os.makedirs(outdir)
print(outdir)
print("------------------------------------")

Etol = 0.5e-6
fmaxzero=0.5e-1

print("----------------------------")
print("## E vs V using 3D unit cell")
if not os.path.exists(outdir):
  os.makedirs(outdir)
  print("## Directory made:",outdir)
pdffil=os.path.join(outdir,'EvsV.pdf')

volumes = np.concatenate((np.arange(860, 1000, 20), np.arange(1000, 1180, 30), np.arange(1180, 1270, 30), np.arange(1270, 1500, 30)))
#volumes = np.append(volumes,1279.129)
#volumes = np.sort(volumes)
energies = []
cifread = read("cof.cif")
Vref = cifread.get_volume()

for vol in volumes:
  tag = f"vol{vol:07.2f}".replace('.', '')
  ciffil=os.path.join(outdir,tag+'.cif')
  print(f"Checking file: {ciffil}")
  datfil=ciffil.replace('.cif','.dat')
  logfil=ciffil.replace('.cif','.log')
  pcklfil=ciffil.replace('.cif','.pckl')
  trajfil=ciffil.replace('.cif','.traj')
  if not os.path.lexists(datfil):
    s=np.power(vol/Vref,1./3.)
    stru=cifread.copy()
    cell=stru.get_cell()
    stru.set_cell([s*cell[0],s*cell[1],s*cell[2]],scale_atoms=True)
    stru.set_calculator(calc)
    stru.set_constraint(FixSymmetry(stru))
    atoms = FrechetCellFilter(stru, constant_volume=True)
    opti = FIRE(atoms, logfile=logfil, trajectory=trajfil, restart=pcklfil)
    Eini = stru.get_potential_energy()
    Epre=Eini
    for i in range(0, 1000, 10):
      opti.run(fmax=fmaxzero, steps=10)
      Ecur = stru.get_potential_energy()
      Edif = Ecur - Epre
      if abs(Edif) < Etol:
        break
      Epre = Ecur
    E = stru.get_potential_energy()
    print("## FIRE lowered energy by {:.6f} eV with final change in energy: {:.6f} eV".format(E-Eini,Edif))
    energies.append(E)
    write(ciffil, stru)
    print(f'## File written:', ciffil)
    symm = check_symmetry(stru, symprec=0.0001, verbose=False)
    fu = stru.get_chemical_formula(empirical=True)
    Z = Formula(str(stru.symbols)).stoichiometry()[2]
    maxforce = np.max(np.abs(stru.get_forces()))
    stress = stru.get_stress() / GPa
    V = stru.get_volume()
    N = len(stru)
    with open(datfil, 'w') as f:
      f.write(f"{V:8.3f} {E:13.6f} ")
      f.write(f"{fu:12s} {Z:2d} {N:5d} ")
      f.write(f"{maxforce:8.4f} ")
      f.write(f"{stress[0]:8.3f} {stress[1]:8.3f} {stress[2]:8.3f} {stress[3]:8.3f} {stress[4]:8.3f} {stress[5]:8.3f} ")
      f.write(f"{symm['number']:3d} ")
      f.write(f"## V(A^3) E(eV) fu Z N maxforce(eV/A) stress(GPa) ITA\n")
    print(f'## File written:', datfil)
  else:
    with open(datfil, 'r') as f:
      lines = f.readlines()
    for line in lines:
      data = line.split()
      if 1e-6 < np.abs(vol - float(data[0])):
        print(f'## ',vol,' .NE.',data[0])
      E = float(data[1])
      energies.append(E)
spline2 = InterpolatedUnivariateSpline(volumes, energies, k=3)
result = minimize_scalar(spline2, bounds=(volumes.min(), volumes.max()), method='bounded')
volume_min = result.x
energy_min = spline2(volume_min)

Vfine = np.linspace(volumes.min(), volumes.max(), 500)
Einterpolated = spline2(Vfine)
cifunicelopt=os.path.join(outdir,'volopt.cif')
datfil=cifunicelopt.replace('.cif','.dat')
if not os.path.lexists(datfil):
  s=np.power(vol/Vref,1./3.)
  unitcell=cifread.copy()
  cell=unitcell.get_cell()
  unitcell.set_cell([s*cell[0],s*cell[1],s*cell[2]],scale_atoms=True)
  unitcell.set_calculator(calc)
  unitcell.set_constraint(FixSymmetry(unitcell))
  atoms = FrechetCellFilter(unitcell, constant_volume=True)
  logfil=cifunicelopt.replace('.cif','.log')
  pcklfil=cifunicelopt.replace('.cif','.pckl')
  trajfil=cifunicelopt.replace('.cif','.traj')
  opti = FIRE(atoms, logfile=logfil, trajectory=trajfil, restart=pcklfil)
  Eini = unitcell.get_potential_energy()
  Epre=Eini
  for i in range(0, 1000, 10):
    opti.run(fmax=fmaxzero, steps=10)
    Ecur = unitcell.get_potential_energy()
    Edif = Ecur - Epre
    if abs(Edif) < Etol:
      break
    Epre = Ecur
  Emin = unitcell.get_potential_energy()
  Vmin = unitcell.get_volume()
  print("## FIRE lowered energy by {:.6f} eV with final change in energy: {:.6f} eV".format(E-Eini,Edif))
  target_z_fractional = 0.5
  cell = unitcell.get_cell()
  inverse_cell = np.linalg.inv(cell)
  write(cifunicelopt,unitcell)
  print("## File written:",cifunicelopt)
  write(cifunicelopt.replace(".cif",".vasp"),unitcell,format='vasp',direct=True)
  print("## File written:",cifunicelopt.replace(".cif",".vasp"))
  symm = check_symmetry(unitcell, symprec=0.0001, verbose=False)
  fu = unitcell.get_chemical_formula(empirical=True)
  Z = Formula(str(unitcell.symbols)).stoichiometry()[2]
  maxforce = np.max(np.abs(unitcell.get_forces()))
  stress = unitcell.get_stress() / GPa
  N = len(unitcell)
  with open(datfil, 'w') as f:
    f.write(f"{Vmin:8.3f} {Emin:13.6f} ")
    f.write(f"{fu:12s} {Z:2d} {N:5d} ")
    f.write(f"{maxforce:8.4f} ")
    f.write(f"{stress[0]:8.3f} {stress[1]:8.3f} {stress[2]:8.3f} {stress[3]:8.3f} {stress[4]:8.3f} {stress[5]:8.3f} ")
    f.write(f"{symm['number']:3d} ")
    f.write(f"## V(A^3) E(eV) fu Z N maxforce(eV/A) stress(GPa) ITA\n")
  print(f'## File written:', datfil)
else:
  with open(datfil, 'r') as f:
    lines = f.readlines()
  for line in lines:
    data = line.split()
    Vmin = float(data[0])
    Emin = float(data[1])

eos = EquationOfState(volumes, energies, eos='murnaghan')
v0, e0, B = eos.fit()
eos.plot(show=True)

bulk_modulus = eos.B / kJ * 1.0e24
bulk_prime = eos.eos_parameters[2]

def pressure(V, V0, K0, K0_prime):
    P = (3 * K0 / 2) * (np.power(V0 / V, 7/3.0) - np.power(V0 / V, 5/3.0)) * \
        (1 + (3/4) * (K0_prime - 4) * (np.power((V0 / V), 2/3) - 1))
    return P

pressures = []

for vol in Vfine:
  P = pressure(vol, eos.v0, bulk_modulus, bulk_prime)
  pressures.append(P)



fig = plt.figure(figsize=(8,6))
gs = gridspec.GridSpec(1, 1, figure=fig)
axi = fig.add_subplot(gs[0, 0])
axi.plot(volumes, energies, 'o', color='teal')
axi.plot(Vfine, Einterpolated, '-', label='M3GNet+D3', color='orange')
axi.plot(volume_min, energy_min, 'o', color='red', label=f'a={volume_min:.5f}')
axi.plot(Vmin, Emin, 'o', color='blue', label=f'a={volume_min:.5f}')
axi.set_xlabel(r'$V$ (\AA$^3$)', fontsize=16)
axi.set_ylabel(r'$E$ (\AA)', fontsize=16)
axi.tick_params(axis='y', labelsize=12)
axi.tick_params(axis='x', labelsize=12)
axi.legend()
plt.tight_layout()
plt.savefig(pdffil)
print('## Figure saved:',pdffil)
plt.show()
print("----------------------------")


{% endhighlight %}







{% highlight ruby%}


import os
import sys

from scipy.constants import angstrom      ## one Angstrom in meters
from scipy.constants import atmosphere    ## standard atmosphere in pascals
from scipy.constants import Boltzmann     ## Boltzmann constant
from scipy.constants import electron_mass ## electron mass
from scipy.constants import electron_volt ## one electron volt in Joules
from scipy.constants import giga
from scipy.constants import hbar          ## h/2pi
from scipy.constants import nano
from scipy.constants import milli
from scipy.constants import physical_constants
from scipy.constants import Planck        ## the Planck constant h

from ase import Atom, Atoms, build
from ase.build import stack, surface
from ase.cell import Cell
from ase.constraints import FixAtoms, FixSymmetry
from ase.dft.kpoints import BandPath
from ase.filters import FrechetCellFilter
from ase.formula import Formula
from ase.geometry import get_distances
from ase.io import read, write
from ase.io.trajectory import Trajectory
from ase.lattice import HEX
from ase.lattice.cubic import FaceCenteredCubic
from ase.optimize import FIRE
from ase.optimize.bfgslinesearch import BFGSLineSearch
from ase.spacegroup.symmetrize import check_symmetry, refine_symmetry
from ase.units import GPa
from ase.visualize import view
import datetime
from importlib.metadata import version, PackageNotFoundError
import importlib.util
from io import StringIO
import numpy as np
from scipy.interpolate import bisplev, InterpolatedUnivariateSpline, RectBivariateSpline
from scipy.optimize import minimize, minimize_scalar

import matplotlib.pyplot as plt
from matplotlib import rc, rcParams
import matplotlib.gridspec as gridspec
rc('text', usetex=True)
plt.rcParams['text.latex.preamble'] = r"\usepackage{amsmath}"

import warnings
import logging
logging.basicConfig(filename='warnings.log', level=logging.WARNING)
logger = logging.getLogger()
def warning_to_log(message, category, filename, lineno, file=None, line=None):
  logger.warning(f'{filename}:{lineno}: {category.__name__}: {message}')
warnings.showwarning = warning_to_log

print("-------------------------------------")
print(f"## Save version info as of {datetime.datetime.now():%Y-%m-%d}")
packages = ["dgl", "pymatgen", "ase", "numpy", "matgl", "dftd3"]
for pkg in packages:
  try:
    pkg_version = version(pkg)
    pkg_location = importlib.util.find_spec(pkg).submodule_search_locations[0]
  except PackageNotFoundError:
    pkg_version = "Not installed"
    pkg_location = "N/A"
  except AttributeError:
    pkg_location = "Location not found"
  print(f"## {pkg:<11} {pkg_version:<10} {pkg_location}")
print("-------------------------------------")

print("------------------------")
print("## Calculator: M3GNet+D3")
from m3gnetd3calculator import m3gnetd3
calc = m3gnetd3()
print("------------------------")

print("------------------------------------")
print("## Output directory <-- sys.argv[1]:",end=' ')
outdir=sys.argv[1]
if not os.path.exists(outdir):
  os.makedirs(outdir)
print(outdir)
print("------------------------------------")

Etol = 0.5e-6
fmaxzero=0.5e-3

print("----------------------------")
print("## E vs V using 3D unit cell")
if not os.path.exists(outdir):
  os.makedirs(outdir)
  print("## Directory made:",outdir)
pdffil=os.path.join(outdir,'EvsV.pdf')
volumes = np.arange(107,116,1)
volumes = np.append(volumes,110.5)
volumes = np.sort(volumes)
energies = []
for vol in volumes:
  tag = f"vol{vol:07.2f}".replace('.', '')
  ciffil=os.path.join(outdir,tag+'.cif')
  datfil=ciffil.replace('.cif','.dat')
  logfil=ciffil.replace('.cif','.log')
  pcklfil=ciffil.replace('.cif','.pckl')
  trajfil=ciffil.replace('.cif','.traj')
  if not os.path.lexists(datfil):
    if os.path.lexists(ciffil):
      stru=read(ciffil)
      print(f'## File read:',ciffil)
    else:
      print(f'## File absent:',ciffil)
      covera = 6.854 / 4.3392
      a=np.power(2*vol/(np.sqrt(3)*covera),1/3.0)
      c=a*covera
      stru = Atoms('BiTeI',
                    cell=HEX(a,c).tocell(),
                    scaled_positions=[(0  , 0  , 0     ),
                                      (2/3, 1/3, 0.6928),
                                      (1/3, 2/3, 0.2510)],
                    pbc=True)
    stru.calc = calc
    stru.set_constraint(FixSymmetry(stru))
    atoms = FrechetCellFilter(stru, constant_volume=True)
    opti = BFGSLineSearch(atoms, logfile=logfil, trajectory=trajfil, restart=pcklfil)
    Eini = stru.get_potential_energy()
    Epre=Eini
    for i in range(0, 1000, 10):
      opti.run(fmax=fmaxzero, steps=10)
      Ecur = stru.get_potential_energy()
      Edif = Ecur - Epre
      if abs(Edif) < Etol:
        break
      Epre = Ecur
    E = stru.get_potential_energy()
    print("## BFGSLineSearch lowered energy by {:.6f} eV with final change in energy: {:.6f} eV".format(E-Eini,Edif))
    energies.append(E)
    write(ciffil, stru)
    print(f'## File written:', ciffil)
    symm = check_symmetry(stru, symprec=0.0001, verbose=False)
    fu = stru.get_chemical_formula(empirical=True)
    Z = Formula(str(stru.symbols)).stoichiometry()[2]
    maxforce = np.max(np.abs(stru.get_forces()))
    stress = stru.get_stress() / GPa
    V = stru.get_volume()
    N = len(stru)
    with open(datfil, 'w') as f:
      f.write(f"{V:8.3f} {E:13.6f} ")
      f.write(f"{fu:12s} {Z:2d} {N:5d} ")
      f.write(f"{maxforce:8.4f} ")
      f.write(f"{stress[0]:8.3f} {stress[1]:8.3f} {stress[2]:8.3f} {stress[3]:8.3f} {stress[4]:8.3f} {stress[5]:8.3f} ")
      f.write(f"{symm['number']:3d} ")
      f.write(f"## V(A^3) E(eV) fu Z N maxforce(eV/A) stress(GPa) ITA\n")
    print(f'## File written:', datfil)
  else:
    with open(datfil, 'r') as f:
      lines = f.readlines()
    for line in lines:
      data = line.split()
      if 1e-6 < np.abs(vol - float(data[0])):
        print(f'## ',vol,' .NE.',data[0])
      E = float(data[1])
      energies.append(E)
spline2 = InterpolatedUnivariateSpline(volumes, energies, k=3)
result = minimize_scalar(spline2, bounds=(volumes.min(), volumes.max()), method='bounded')
volume_min = result.x
energy_min = spline2(volume_min)

Vfine = np.linspace(volumes.min(), volumes.max(), 500)
Einterpolated = spline2(Vfine)
cifunicelopt=os.path.join(outdir,'volopt.cif')
datfil=cifunicelopt.replace('.cif','.dat')
if not os.path.lexists(datfil):
  if os.path.lexists(cifunicelopt):
    unitcell=read(cifunicelopt)
    print(f'## File read:',cifunicelopt)
  else:
    covera = 6.854 / 4.3392
    a=np.power(2*volume_min/(np.sqrt(3)*covera),1/3.0)
    c=a*covera
    unitcell = Atoms('BiTeI',
                              cell=HEX(a,c).tocell(),
                              scaled_positions=[(0  , 0  , 0     ),
                                                (2/3, 1/3, 0.6928),
                                                (1/3, 2/3, 0.2510)],
                              pbc=True)
  unitcell.calc = calc
  unitcell.set_constraint(FixSymmetry(unitcell))
  atoms = FrechetCellFilter(unitcell, constant_volume=True)
  logfil=cifunicelopt.replace('.cif','.log')
  pcklfil=cifunicelopt.replace('.cif','.pckl')
  trajfil=cifunicelopt.replace('.cif','.traj')
  opti = BFGSLineSearch(atoms, logfile=logfil, trajectory=trajfil, restart=pcklfil)
  Eini = unitcell.get_potential_energy()
  Epre=Eini
  for i in range(0, 1000, 10):
    opti.run(fmax=fmaxzero, steps=10)
    Ecur = unitcell.get_potential_energy()
    Edif = Ecur - Epre
    if abs(Edif) < Etol:
      break
    Epre = Ecur
  Emin = unitcell.get_potential_energy()
  Vmin = unitcell.get_volume()
  print("## BFGSLineSearch lowered energy by {:.6f} eV with final change in energy: {:.6f} eV".format(E-Eini,Edif))
  target_z_fractional = 0.5
  cell = unitcell.get_cell()
  inverse_cell = np.linalg.inv(cell)
  for atom in unitcell:
    if atom.symbol == 'Bi':
      current_position = atom.position
      current_fractional_position = np.dot(current_position, inverse_cell.T)
      current_z_fractional = current_fractional_position[2]
      shift_fractional = target_z_fractional - current_z_fractional
      shift_cartesian = shift_fractional * cell[:, 2]
      break
  unitcell.translate(shift_cartesian)
  write(cifunicelopt,unitcell)
  print("## File written:",cifunicelopt)
  write(cifunicelopt.replace(".cif",".vasp"),unitcell,format='vasp',direct=True)
  print("## File written:",cifunicelopt.replace(".cif",".vasp"))
  symm = check_symmetry(unitcell, symprec=0.0001, verbose=False)
  fu = unitcell.get_chemical_formula(empirical=True)
  Z = Formula(str(unitcell.symbols)).stoichiometry()[2]
  maxforce = np.max(np.abs(unitcell.get_forces()))
  stress = unitcell.get_stress() / GPa
  N = len(unitcell)
  with open(datfil, 'w') as f:
    f.write(f"{Vmin:8.3f} {Emin:13.6f} ")
    f.write(f"{fu:12s} {Z:2d} {N:5d} ")
    f.write(f"{maxforce:8.4f} ")
    f.write(f"{stress[0]:8.3f} {stress[1]:8.3f} {stress[2]:8.3f} {stress[3]:8.3f} {stress[4]:8.3f} {stress[5]:8.3f} ")
    f.write(f"{symm['number']:3d} ")
    f.write(f"## V(A^3) E(eV) fu Z N maxforce(eV/A) stress(GPa) ITA\n")
  print(f'## File written:', datfil)
else:
  with open(datfil, 'r') as f:
    lines = f.readlines()
  for line in lines:
    data = line.split()
    Vmin = float(data[0])
    Emin = float(data[1])
fig = plt.figure(figsize=(8,6))
gs = gridspec.GridSpec(1, 1, figure=fig)
axi = fig.add_subplot(gs[0, 0])
axi.plot(volumes, energies, 'o', color='teal')
axi.plot(Vfine, Einterpolated, '-', label='Spline interpolation', color='orange')
axi.plot(volume_min, energy_min, 'o', color='red', label=f'a={volume_min:.5f}')
axi.plot(Vmin, Emin, 'o', color='blue', label=f'a={volume_min:.5f}')
axi.set_xlabel(r'$V$ (\AA$^3$)', fontsize=16)
axi.set_ylabel(r'$E$ (\AA)', fontsize=16)
axi.tick_params(axis='y', labelsize=12)
axi.tick_params(axis='x', labelsize=12)
axi.legend()
plt.tight_layout()
plt.savefig(pdffil)
print('## Figure saved:',pdffil)
plt.show()
print("----------------------------")


{% endhighlight %}






{% highlight ruby%}

import os
import sys

from scipy.constants import angstrom      ## one Angstrom in meters
from scipy.constants import atmosphere    ## standard atmosphere in pascals
from scipy.constants import Boltzmann     ## Boltzmann constant
from scipy.constants import electron_mass ## electron mass
from scipy.constants import electron_volt ## one electron volt in Joules
from scipy.constants import giga
from scipy.constants import hbar          ## h/2pi
from scipy.constants import nano
from scipy.constants import milli
from scipy.constants import physical_constants
from scipy.constants import Planck        ## the Planck constant h
from scipy.optimize import curve_fit

import sympy as sp

from ase import Atom, Atoms, build
from ase.build import stack, surface
from ase.cell import Cell
from ase.constraints import FixAtoms, FixSymmetry
from ase.dft.kpoints import BandPath
from ase.filters import FrechetCellFilter
from ase.formula import Formula
from ase.geometry import get_distances
from ase.io import read, write
from ase.io.trajectory import Trajectory
from ase.lattice import HEX
from ase.lattice.cubic import FaceCenteredCubic
from ase.optimize import FIRE
from ase.optimize.bfgslinesearch import BFGSLineSearch
from ase.spacegroup.symmetrize import check_symmetry, refine_symmetry
from ase.units import GPa
from ase.units import kJ
from ase.build import bulk
from ase.calculators.emt import EMT
from ase.eos import calculate_eos
from ase.visualize import view
import datetime
from importlib.metadata import version, PackageNotFoundError
import importlib.util
from io import StringIO
from ase.eos import EquationOfState
import numpy as np
from scipy.interpolate import bisplev, InterpolatedUnivariateSpline, RectBivariateSpline
from scipy.optimize import minimize, minimize_scalar
import math
import matplotlib.pyplot as plt
from matplotlib import rc, rcParams
import matplotlib.gridspec as gridspec

rc('text', usetex=True)
plt.rcParams['text.latex.preamble'] = r"\usepackage{amsmath}"

import warnings
import logging
logging.basicConfig(filename='warnings.log', level=logging.WARNING)
logger = logging.getLogger()
def warning_to_log(message, category, filename, lineno, file=None, line=None):
  logger.warning(f'{filename}:{lineno}: {category.__name__}: {message}')
warnings.showwarning = warning_to_log

print("-------------------------------------")
print(f"## Save version info as of {datetime.datetime.now():%Y-%m-%d}")
packages = ["dgl", "pymatgen", "ase", "numpy", "matgl", "dftd3"]
for pkg in packages:
  try:
    pkg_version = version(pkg)
    pkg_location = importlib.util.find_spec(pkg).submodule_search_locations[0]
  except PackageNotFoundError:
    pkg_version = "Not installed"
    pkg_location = "N/A"
  except AttributeError:
    pkg_location = "Location not found"
  print(f"## {pkg:<11} {pkg_version:<10} {pkg_location}")
print("-------------------------------------")


print("------------------------")
print("## Calculator: M3GNet+D3")
from m3gnetd3calculator import m3gnet, m3gnetd3, chgnet
calc = m3gnetd3()
calcd3 = m3gnetd3()
calc_chg = chgnet()
print("------------------------")

calculators = [calc, calcd3, calc_chg]

print("------------------------------------")
print("## Output directory <-- sys.argv[1]:",end=' ')
outdir=sys.argv[1]
if not os.path.exists(outdir):
  os.makedirs(outdir)
print(outdir)
print("------------------------------------")

Etol = 0.5e-6
fmaxzero=0.5e-1

print("----------------------------")
print("## E vs V using 3D unit cell")
if not os.path.exists(outdir):
  os.makedirs(outdir)
  print("## Directory made:",outdir)
pdffil=os.path.join(outdir,'EvsV.pdf')

volumes = np.concatenate((np.arange(860, 1000, 20), np.arange(1000, 1180, 30), np.arange(1180, 1270, 30), np.arange(1270, 1500, 30)))
#volumes = np.append(volumes,1279.129)
#volumes = np.sort(volumes)
energies = []
cifread = read("cof.cif")
Vref = cifread.get_volume()
for vol in volumes:
  tag = f"vol{vol:07.2f}".replace('.', '')
  ciffil=os.path.join(outdir,tag+'.cif')
  print(f"Checking file: {ciffil}")
  datfil=ciffil.replace('.cif','.dat')
  logfil=ciffil.replace('.cif','.log')
  pcklfil=ciffil.replace('.cif','.pckl')
  trajfil=ciffil.replace('.cif','.traj')
  if not os.path.lexists(datfil):
    s=np.power(vol/Vref,1./3.)
    stru=cifread.copy()
    cell=stru.get_cell()
    stru.set_cell([s*cell[0],s*cell[1],s*cell[2]],scale_atoms=True)
    stru.set_calculator(calc)
    stru.set_constraint(FixSymmetry(stru))
    atoms = FrechetCellFilter(stru, constant_volume=True)
    opti = FIRE(atoms, logfile=logfil, trajectory=trajfil, restart=pcklfil)
    Eini = stru.get_potential_energy()
    Epre=Eini
    for i in range(0, 1000, 10):
      opti.run(fmax=fmaxzero, steps=10)
      Ecur = stru.get_potential_energy()
      Edif = Ecur - Epre
      if abs(Edif) < Etol:
        break
      Epre = Ecur
    E = stru.get_potential_energy()
    print("## FIRE lowered energy by {:.6f} eV with final change in energy: {:.6f} eV".format(E-Eini,Edif))
    energies.append(E)
    write(ciffil, stru)
    print(f'## File written:', ciffil)
    symm = check_symmetry(stru, symprec=0.0001, verbose=False)
    fu = stru.get_chemical_formula(empirical=True)
    Z = Formula(str(stru.symbols)).stoichiometry()[2]
    maxforce = np.max(np.abs(stru.get_forces()))
    stress = stru.get_stress() / GPa
    V = stru.get_volume()
    N = len(stru)
    with open(datfil, 'w') as f:
      f.write(f"{V:8.3f} {E:13.6f} ")
      f.write(f"{fu:12s} {Z:2d} {N:5d} ")
      f.write(f"{maxforce:8.4f} ")
      f.write(f"{stress[0]:8.3f} {stress[1]:8.3f} {stress[2]:8.3f} {stress[3]:8.3f} {stress[4]:8.3f} {stress[5]:8.3f} ")
      f.write(f"{symm['number']:3d} ")
      f.write(f"## V(A^3) E(eV) fu Z N maxforce(eV/A) stress(GPa) ITA\n")
    print(f'## File written:', datfil)
  else:
    with open(datfil, 'r') as f:
      lines = f.readlines()
    for line in lines:
      data = line.split()
      if 1e-6 < np.abs(vol - float(data[0])):
        print(f'## ',vol,' .NE.',data[0])
      E = float(data[1])
      energies.append(E)
spline2 = InterpolatedUnivariateSpline(volumes, energies, k=3)
result = minimize_scalar(spline2, bounds=(volumes.min(), volumes.max()), method='bounded')
volume_min = result.x
energy_min = spline2(volume_min)

Vfine = np.linspace(volumes.min(), volumes.max(), 500)
Einterpolated = spline2(Vfine)
cifunicelopt=os.path.join(outdir,'volopt.cif')
datfil=cifunicelopt.replace('.cif','.dat')
if not os.path.lexists(datfil):
  s=np.power(vol/Vref,1./3.)
  unitcell=cifread.copy()
  cell=unitcell.get_cell()
  unitcell.set_cell([s*cell[0],s*cell[1],s*cell[2]],scale_atoms=True)
  unitcell.set_calculator(calc)
  unitcell.set_constraint(FixSymmetry(unitcell))
  atoms = FrechetCellFilter(unitcell, constant_volume=True)
  logfil=cifunicelopt.replace('.cif','.log')
  pcklfil=cifunicelopt.replace('.cif','.pckl')
  trajfil=cifunicelopt.replace('.cif','.traj')
  opti = FIRE(atoms, logfile=logfil, trajectory=trajfil, restart=pcklfil)
  Eini = unitcell.get_potential_energy()
  Epre=Eini
  for i in range(0, 1000, 10):
    opti.run(fmax=fmaxzero, steps=10)
    Ecur = unitcell.get_potential_energy()
    Edif = Ecur - Epre
    if abs(Edif) < Etol:
      break
    Epre = Ecur
  Emin = unitcell.get_potential_energy()
  Vmin = unitcell.get_volume()
  print("## FIRE lowered energy by {:.6f} eV with final change in energy: {:.6f} eV".format(E-Eini,Edif))
  target_z_fractional = 0.5
  cell = unitcell.get_cell()
  inverse_cell = np.linalg.inv(cell)
  write(cifunicelopt,unitcell)
  print("## File written:",cifunicelopt)
  write(cifunicelopt.replace(".cif",".vasp"),unitcell,format='vasp',direct=True)
  print("## File written:",cifunicelopt.replace(".cif",".vasp"))
  symm = check_symmetry(unitcell, symprec=0.0001, verbose=False)
  fu = unitcell.get_chemical_formula(empirical=True)
  Z = Formula(str(unitcell.symbols)).stoichiometry()[2]
  maxforce = np.max(np.abs(unitcell.get_forces()))
  stress = unitcell.get_stress() / GPa
  N = len(unitcell)
  with open(datfil, 'w') as f:
    f.write(f"{Vmin:8.3f} {Emin:13.6f} ")
    f.write(f"{fu:12s} {Z:2d} {N:5d} ")
    f.write(f"{maxforce:8.4f} ")
    f.write(f"{stress[0]:8.3f} {stress[1]:8.3f} {stress[2]:8.3f} {stress[3]:8.3f} {stress[4]:8.3f} {stress[5]:8.3f} ")
    f.write(f"{symm['number']:3d} ")
    f.write(f"## V(A^3) E(eV) fu Z N maxforce(eV/A) stress(GPa) ITA\n")
  print(f'## File written:', datfil)
else:
  with open(datfil, 'r') as f:
    lines = f.readlines()
  for line in lines:
    data = line.split()
    Vmin = float(data[0])
    Emin = float(data[1])

eos = EquationOfState(volumes, energies, eos='murnaghan')
v0, e0, B = eos.fit()
eos.plot(show=True)

bulk_modulus = eos.B / kJ * 1.0e24
bulk_prime = eos.eos_parameters[2]

def pressure(V, V0, K0, K0_prime):
    P = (3 * K0 / 2) * (np.power(V0 / V, 7/3.0) - np.power(V0 / V, 5/3.0)) * \
        (1 + (3/4) * (K0_prime - 4) * (np.power((V0 / V), 2/3) - 1))
    return P

pressures = []

for vol in Vfine:
  P = pressure(vol, eos.v0, bulk_modulus, bulk_prime)
  pressures.append(P)



fig = plt.figure(figsize=(8,6))
gs = gridspec.GridSpec(1, 1, figure=fig)
axi = fig.add_subplot(gs[0, 0])
axi.plot(volumes, energies, 'o', color='teal')
axi.plot(Vfine, Einterpolated, '-', label='M3GNet+D3', color='orange')
axi.plot(volume_min, energy_min, 'o', color='red', label=f'a={volume_min:.5f}')
axi.plot(Vmin, Emin, 'o', color='blue', label=f'a={volume_min:.5f}')
axi.set_xlabel(r'$V$ (\AA$^3$)', fontsize=16)
axi.set_ylabel(r'$E$ (\AA)', fontsize=16)
axi.tick_params(axis='y', labelsize=12)
axi.tick_params(axis='x', labelsize=12)
axi.legend()
plt.tight_layout()
plt.savefig(pdffil)
print('## Figure saved:',pdffil)
plt.show()
print("----------------------------")


{% endhighlight %}



