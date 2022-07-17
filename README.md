# circom 7 snarkjs Commands used for creating files required for zksnark project

### commands for groth16 & js files.

``
rm circuit.r1cs
rm circuit.sym
rm circuit_*
rm -r circuit_*
rm powers*
rm proof.json
rm public.json
rm verifier.sol
rm verification_key.json
rm witness.wtns
rm parameters.txt
``

#folder has circom.circom
#folder has input.json

circom circom.circom --r1cs --wasm --sym --c

### above compile command, at root level creates circuit.r1cs circuit.sym 
### above compile command creates circuit_js (has circuit.wasm, generate_witness.js & other files)


cd circuit_js

### generate witness file at root level
node generate_witness.js circuit.wasm ../input.json ../witness.json

### Now we will start using snrakjs to generate & validate proof
### groth16 zk-snark requires to creates phase 1 & phase 2 trusted setup

### Phase 1. Powers of tau

### create new powersoftau ceremony
snarkjs powersoftau new bn128 12 powers12_0000.ptau -v

### contribute to the created ceremony
snarkjs powersoftau contribute powers12_0000.ptau powers12_0001.ptau --name="first contribution to ceremony" -v -e="first contribution some random text"

### Phase 2 of etup ceremony
snarkjs powersoftau prepare phase2 powers12_0001.ptau powers12_final.ptau -v

### Generating zkey file for proving & verification 
snarkjs groth16 setup circuit.r1cs powers12_final.ptau circuit_0000.zkey

### adding contribution to zkey
snarkjs zkey contribute circuit_0000.zkey circuit_0001.zkey --name="1st Contributor Name" -v

### The zkey contribute command creates a zkey file with a new contribution.

### Export the verification key
snarkjs zkey export verificationkey circuit_0001.zkey verification_key.json

### Create the proof
snarkjs groth16 prove circuit_0001.zkey witness.wtns proof.json public.json

### proof.json contains the actual proof
### public.json contains the values of the public inputs and output.

### Verify the proof
snarkjs plonk verify verification_key.json public.json proof.json

### Turn the verifier into a smart contract
snarkjs zkey export solidityverifier circuit_final.zkey verifier.sol

### Simulate a verification call
### can be used on remix ide
snarkjs zkey export soliditycalldata public.json proof.json
