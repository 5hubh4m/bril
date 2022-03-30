
This work is by [@5hubh4m](github.com/5hubh4m), [@ayakayorihiro](github.com/ayakayorihiro), and [@anshumanmohan](github.com/anshumanmohan).

## Implementation

We implemented a straightforward stop-the-world tracing collector with one space and no compaction. We found this assignment relatively manageable. Our work is in two broad parts:

### Tweaks to Memory

We need a few edits to the memory model to make it amenable to garbage collection. 

The main change is in `Env`, which goes from  
`type Env = Map<bril.Ident, Value>;`  
to  
`type Env = Map<bril.Ident, Value>[];`.  
That is to say, an array of maps that simulates the call stack.
This creates some straightforward cleanup downstream.

### The Actual Collection

A `Heap` contains  
`private readonly storage: Map<number, Value[]>`  
and a `Value` is an ADT:  
`Value = boolean | BigInt | Pointer | number;`  
This means that we need not be conservative; we can precisely follow `Pointer`s to check for reachability. 

We change the existing `free` operation into a no-op. Every 10,000 instructions trigger a collection, and the end of the main function also triggers a collection. A collection is a `trace`, which lists everything that is to be freed, followed by `collect`, which essentially calls the dearly departed `free` operation on each of these. The meat of the matter is clearly `trace`, so we discuss it a little further.

The method `trace` identifies roots by walking over the given environment and picking out values that were made with the `Pointer` constructor. These go into the "grey" set. Everything else goes into the "white" set. We follow a worklist strategy, iterating over the grey locations and promoting locations from white to grey and grey to black in the usual way. This buys us recursive power without some of the hassle of actual recursion.

## Testing

We tested our work on the bril examples in:
1. the benchmarks directory
2. the test/mem directory

Of course, only a small number of the benchmarks use the memory extension. We found that outputs were unchanged, which is good, and that no memory leaks were introduced, which is better.

## Challenges

We thought this went pretty quickly. The main challenge was getting used to some of the funkier syntax and language decisions that the TypeScript language makes.