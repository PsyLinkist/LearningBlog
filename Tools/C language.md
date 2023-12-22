## Struct
### Storage of struct
Size of a struct is not simply defined by its size of fields. Every fields in the struct has its **number of alignments**, rules below:  
1. The first field is aligned to the start of the struct;
2. Alignment of a field = `min(sizeof(field), definition in compiler)`;
3. The last field is aligned to the maximum alignment in the struct.

### Why alignment?
- Hardware awareness;
- Optimization. Cpu has to access memory twice if not aligned.
TODO: But why? How does cpu work?