## Simulating the Stokes equation in python 

### Installation:
```
python setup.py install

```

### Example 1 : computing rigid body motion of particles.

```
import pystokes
import pyforces
import numpy as np

a, Np, dim = 1, 3, 3                 # radius and number of particles and dimension
v = np.zeros(dim*Np)                 # Memory allocation for velocity
r = np.zeros(dim*Np)                 # Position vector of the particles
F = np.zeros(dim*Np)                 # Forces on the particles

r[0], r[1], r[2] = -4, 0 , 4         # x-comp of PV of particles

pRbm = pystokes.periodic.Rbm(a, Np, L)    # instantiate the classes
ff = pyforces.ForceFields.Forces(Np)

print 'Initial velocity', v

ff.sedimentation(F, g=10)            # call the Sedimentation module of pyforces
pRbm.stokesletV(v, r, F)               # and StokesletV module of pystokes

print 'Updated velocity', v
```

### Example 2 : computing flow due to particles.
```
import pystokes
import pyforces
import numpy as np

a, Np, dim = 1, 1000, 3              # Radius, No of particle and dimension
S0, D0 = 1, 1                        # Multipole Strength 

r = np.zeros(dim*Np)                 # position vector of particles
p = np.zeros(dim*Np)                 # orientation vector of particles
v = np.zeros(dim*Np)                 # velocity vector of particles
F = np.zeros(dim*Np)                 # forces on particles
S = np.zeros(5*Np)                   # stresslets on particles
D = np.zeros(dim*Np)                 # potential dipoles on particles

Nt = 10                              # number of field points
rt = np.random.rand(dim*Nt)*100      # field point locations
vt = np.zeros(dim*Nt)                # velocity at field points


flwpnts = pystokes.periodic.Flow(a, Np, Nt) # instantiate the classes
ff = pyforces.ForceFields.Forces(Np)

r[0: Np] = np.linspace(0,3*Np,Np)    # distribute particles on a line along the x-axis 
ff.sedimentation(F, g=10)            # add forces in the z-direction

p[0: Np] = 1                         # orient particles along x-axis

# parametrize the stresslets uni-axially
S[    0:  Np] = S0*(p[0:Np]*p[0:Np] - 1/3)
S[  Np :2*Np] = S0*(p[Np:2*Np]*p[Np:2*Np] - 1/3)
S[2*Np :3*Np] = S0*(p[0:Np]*p[Np:2*Np])
S[3*Np :4*Np] = S0*(p[0:Np]*p[2*Np:3*Np])
S[4*Np :5*Np] = S0*(p[Np:2*Np]*p[2*Np:3*Np])

# parametrize the dipoles uniaxially
D[::] = D0*p[::]

print 'Initial flow at rt' vt

flwpnts.stokesletV(vt, rt, r, F)     # add flow due to stokeslet
flwpnts.stressletV(vt, rt, r, S)     # add flow due to stresslet
flwpnts.potDipoleV(vt, rt, r, D)     # add flow due to potential dipole

print 'Updated flow at rt' vt
```


### Example 3 : Grid based simulation Technique.
```
import pystokes
import pyforces
import numpy as np

Lx, Ly, Lz  = 128, 128, 128          # extent of the box
Nx, Ny, Nz  = 128, 128, 128          # grid division 

a, Np, dim = 1, 1000, 3              # Radius, No of particle and dimension
S0, D0 = 1, 1                        # Multipole Strength 

r = np.zeros(dim*Np)                 # position vector of particles
V = np.zeros(dim*Np)                 # Velocity vector of particles
p = np.zeros(dim*Np)                 # orientation vector of particles
F = np.zeros(dim*Np)                 # forces on particles
S = np.zeros(5*Np)                   # stresslets on particles
D = np.zeros(dim*Np)                 # potential dipoles on particles
vv = np.zeros(dim*Nx*Ny*Nz)          # velocity at grid points

pmima = pystokes.mima.flow(a, Np, Lx, Ly, Lz, Nx, Ny, Nz)   # instantiate the classes
ff = pyforces.ForceFields.Forces(Np)

ff.sedimentation(F, g=10)                                   # call the Sedimentation module of pyforces
pmima.stokesletV(vv, r, F, sigma = 3.0, NN = 3*sigma)       # update grid velocity

print 'Initial velocity', V

pmima.interpolate(V, r, vv, sigma = 3.0, NN = 3*sigma)      # interpolate back velocity of the particles

print 'Final velocity', V
```