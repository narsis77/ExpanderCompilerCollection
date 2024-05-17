# Artifact and Serialization

There are two main compilation artifacts: layered circuit and input solver.

Layered circuit is the final form of the circuit, it will be used by the prover and verifier.

Input solver is a middle form of the circuit, it will be used to generate the witness (input).

## Layered Circuit Format

Layered Circuits are defined in `layered/circuit.go`.

### Introduction

We denote the whole circuit as `RootCircuit`, and each layer of the circuit as `Circuit`.

`RootCircuit` contains many layers, each layers contains 2^n gates. `Circuit` saves the wiring of the gates between two adjacent layers.

A `Circuit` may contain other `Circuit` as sub-circuit.

The ID of a `Circuit` is its index in `RootCircuit.Circuits`.

`RootCircuit.Layers` saves the IDs of the `Circuit` of each layer.

### Special Internal Representation

If `Coef` equals to `RootCircuit.Field`, it's a random gate.

### Serialization

Basically, `uint64` is serialized as little endian, and `big.Int` is serialized as 32-byte little endian.

Arrays are presented by a `uint64` length, followed by the serialization of elements.

The only difference between serialization and internal representation is: `Coef` are always less than `RootCircuit.Field` in serialized form. And we use additional arrays to present random gates.

Here the serialzied `Circuit` struct can be view as:

```go
type Circuit struct {
	InputLen      uint64
	OutputLen     uint64
	SubCircuits   []SubCircuit
	Mul           []GateMul
	Add           []GateAdd
	Cst           []GateCst
    RandomCoefIdx []uint64
}
```

where `RandomCoefIdx` saves the index of the random gates in `Mul+Add+Cst`.

Finally, there's a magic uint64 number 3626604230490605891 (`b'CIRCUIT2'`) at the beginning of the serialized `RootCircuit`.

## Input Solver Format

Input solver is defined in `ir/input_solver.go`.

Since it should be only used by go, the serialization is done by gob.

## Witness Format

Witness is a `big.Int` array, it's the input of the layered circuit. It's also defined in `ir/input_solver.go`.

The serialization is really simple. Since we already know the length of the array, we can just serialize the `big.Int` array as 32-byte little endian.