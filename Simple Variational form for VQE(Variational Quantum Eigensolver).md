to prepare the variational form one needs an anstaz or an arbitary ground state to start with, and a parameterized circuit that runs on the initial state to set parameters 
there are two types of parameterized circuits:


## 1-General circuits like RyRz and Ry 
Those circuits are multilayers of gates with certain depth and other configurations like numbers of qubits and entagelement settings.. where a "layer" refers to the number of repetition of set of gates and the depth is "the maximum time from input to output assuming all gates take 1 unit of time." the more gates we add the more parameters we have and thus the we have a bigger set of anstaz to make. 


## 2-Domain specific circuits like UCCSD 
restrict the parameters in a circuit with a known property of the system like number of particles or like a physical property of the simulation system ex: "set of circuits realizable on real quantum hardware"

in this code we impelement one of the circuits of the seconed type which solves the ground state of hamiltonian for a 2 qubit system.
we make measurement and then change the parameters of the circuit (angle1) to see how the output differes

through fermion qubit mapping we got a hamiltonian: 
h2_hamiltonian = (-1.0523732)  II + 
                 (0.39793742)  IZ + 
                 (-0.3979374)  ZI + 
                 (-0.0112801)  ZZ + 
                 (0.18093119)  XX



```python
from numpy import pi
from qiskit import QuantumCircuit, Aer, execute
from qiskit.visualization import plot_histogram
```


```python
def para_h2_circuit(depth,angle1,angle2):
    para_circuit=QuantumCircuit(depth)
    para_circuit.ry(angle1,0)
    para_circuit.ry(angle1,1)
    para_circuit.rz(angle1,0)
    para_circuit.rz(angle1,1)
    
    for i in range(depth):
        para_circuit.cx(0,1)
        para_circuit.ry(angle2,0)
        para_circuit.rz(angle2,0)
        para_circuit.ry(angle2,1)
        para_circuit.rz(angle2,1)
    return para_circuit
```


```python
VQE_circuit1=para_h2_circuit(2,pi/2,pi/2)
VQE_circuit1.draw()
```




<pre style="word-wrap: normal;white-space: pre;background: #fff0;line-height: 1.1;font-family: &quot;Courier New&quot;,Courier,monospace">     ┌──────────┐┌──────────┐     ┌──────────┐┌──────────┐     ┌──────────┐»
q_0: ┤ RY(pi/2) ├┤ RZ(pi/2) ├──■──┤ RY(pi/2) ├┤ RZ(pi/2) ├──■──┤ RY(pi/2) ├»
     ├──────────┤├──────────┤┌─┴─┐├──────────┤├──────────┤┌─┴─┐├──────────┤»
q_1: ┤ RY(pi/2) ├┤ RZ(pi/2) ├┤ X ├┤ RY(pi/2) ├┤ RZ(pi/2) ├┤ X ├┤ RY(pi/2) ├»
     └──────────┘└──────────┘└───┘└──────────┘└──────────┘└───┘└──────────┘»
«     ┌──────────┐
«q_0: ┤ RZ(pi/2) ├
«     ├──────────┤
«q_1: ┤ RZ(pi/2) ├
«     └──────────┘</pre>



## we will calculate the energy of the circuit  


```python
simulator=Aer.get_backend('qasm_simulator')

meas=VQE_circuit1.copy()
meas.measure_all()
result=execute(meas, backend=simulator,shots=10000).result()
counts=result.get_counts()
print(counts)

```

    {'10': 2527, '00': 2381, '01': 2517, '11': 2575}
    


```python
def ZZ_measurement(circuit,counts):

    if '00' not in counts:
        counts['00'] = 0
    if '01' not in counts:
        counts['01'] = 0
    if '10' not in counts:
        counts['10'] = 0
    if '11' not in counts:
        counts['11'] = 0 



    total_counts = counts['00'] + counts['11'] + counts['01'] + counts['10']
    zz = counts['00'] + counts['11'] - counts['01'] - counts['10']
    zz = zz / total_counts
    
    return zz
    
        
```


```python
def ZI_measurement(circuit,counts):
    
    if '00' not in counts:
        counts['00'] = 0
    if '01' not in counts:
        counts['01'] = 0
    if '10' not in counts:
        counts['10'] = 0
    if '11' not in counts:
        counts['11'] = 0 



    total_counts = counts['00'] + counts['11'] + counts['01'] - counts['10']
    zi = counts['00'] - counts['11'] + counts['01'] + counts['10']
    zi = zi / total_counts
    
    return zi
    
        
```


```python
def IZ_measurement(circuit,counts):

    if '00' not in counts:
        counts['00'] = 0
    if '01' not in counts:
        counts['01'] = 0
    if '10' not in counts:
        counts['10'] = 0
    if '11' not in counts:
        counts['11'] = 0 



    total_counts = counts['00'] + counts['11'] + counts['01'] + counts['10']
    iz = counts['00'] - counts['11'] - counts['01'] + counts['10']
    iz = iz / total_counts
    
    return iz
    
        
```


```python
def XX_measurement(circuit):
    xx_meas = circuit.copy()
    xx_meas.h(0)
    xx_meas.h(1)
    xx_meas.measure_all()
    result=execute(xx_meas, backend=simulator,shots=10000).result()
    counts=result.get_counts()
    
    if '00' not in counts:
        counts['00'] = 0
    if '01' not in counts:
        counts['01'] = 0
    if '10' not in counts:
        counts['10'] = 0
    if '11' not in counts:
        counts['11'] = 0 


    total_counts = counts['00']+ counts['11'] + counts['01'] + counts['10']
    xx = counts['00'] + counts['11'] - counts['01'] - counts['10']
    xx = xx / total_counts
    return xx
```


```python
xx=XX_measurement(VQE_circuit1)
print(xx)
```

    -1.0
    


```python
def get_energy(circuit,meas_circuit,counts):
    
    zz = ZZ_measurement(meas_circuit,counts)
    iz = IZ_measurement(meas_circuit,counts)
    zi = ZI_measurement(meas_circuit,counts)
    xx = XX_measurement(circuit)

    energy = (-1.0523732)*1 + (0.39793742)*iz + (-0.3979374)*zi + (-0.0112801)*zz + (0.18093119)*xx
    
    return energy
```

# solving for energy 


```python
energy=get_energy(VQE_circuit1,meas,counts)
print(energy)
```

    -1.6307407583629208
    

# Changing the parameters of paramtrized circuit in 2 different directions 


```python
VQE_circuit_plus=para_h2_circuit(2, pi/2 + 0.1*pi/2, pi/2)
meas2=VQE_circuit_plus.copy()
meas2.measure_all()
result=execute(meas2, backend=simulator,shots=10000).result()
counts2=result.get_counts()
print(counts2)
energy_plus=get_energy(VQE_circuit_plus,meas2,counts2)
print(energy_plus)
```

    {'10': 2513, '00': 2553, '01': 1717, '11': 3217}
    -1.5108836223956477
    


```python
VQE_circuit_minus=para_h2_circuit(2, pi/2 - 0.1*pi/2, pi/2)
meas3=VQE_circuit_minus.copy()
meas3.measure_all()
result=execute(meas3, backend=simulator,shots=10000).result()
counts3=result.get_counts()
print(counts3)
energy_minus=get_energy(VQE_circuit_minus,meas3,counts3)
print(energy_minus)
```

    {'10': 2539, '00': 2538, '01': 3158, '11': 1765}
    -1.744280205366321
    

## it looks like the direction of decrease of energy is the positive direction


```python

```
