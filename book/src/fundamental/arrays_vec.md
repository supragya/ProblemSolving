# Arrays and Vectors
Arrays and Vector based problems are unique in the sense that they are often used
as screening interview problem statements.

## Important differentiators
Arrays in rust / any language are supposed to be elements whose size is known at
compile time, hence `[T;N]` where `T` is some type and `N` is the number of elements.

Arrays implement `Sized` trait, their size is known at compile time.

Vectors on the other hand, store things on heap and not the stack. So, they are not
technically `Sized` in their contents, but they are `Sized` since all a rust Vector
is is a pointer to memory, the length of contents and capacity.

```rust
fn takes_only_sized(only_sized: impl Sized) {
    println!("Size: {}", std::mem::size_of_val(&only_sized));
}
fn main() {
    println!("CPU word size: {}", std::mem::size_of::<usize>());

    let array = [9, 1, 2, 3];
    takes_only_sized(array);

    let vector = Vec::<u32>::new();
    takes_only_sized(vector);
}
```

## Key operations on arrays and vectors
All arrays and vectors implement conversion to a slice. In rust, a slice is
a dynamically-sized view into a contiguous sequence. Slices are very useful since
they provide interesting methods that allow for split borrowing among other things!



### Splitting

### Chunking
Chunking operations allow you to create smaller chunks from a larger slice. Below is an
example:
```rust
let slice = ['l', 'o', 'r', 'e', 'm'];
let mut iter = slice.chunks(2);
assert_eq!(iter.next().unwrap(), &['l', 'o']);
assert_eq!(iter.next().unwrap(), &['r', 'e']);
assert_eq!(iter.next().unwrap(), &['m']);
assert!(iter.next().is_none());

```
Chunking is especially useful in cases where we need parallel processing via threads etc.

#### Constant sized chunking
Here are a few import functions to know for constant sized chunking:
```rust,ignore
// For Some(&['l', 'o']), Some(&['r', 'e']), Some(&['m']) 
pub fn chunks(&self, chunk_size: usize) -> Chunks<'_, T>

// For Some(&mut ['l', 'o']), Some(&mut ['r', 'e']), Some(&mut ['m']) 
pub fn chunks_mut(&mut self, chunk_size: usize) -> ChunksMut<'_, T>

// For Some(&['l', 'o']), Some(&['r', 'e']), None 
pub fn chunks_exact(&self, chunk_size: usize) -> ChunksExact<'_, T>

// For Some(&mut ['l', 'o']), Some(&mut ['r', 'e']), None 
pub fn chunks_exact_mut(&mut self, chunk_size: usize) -> ChunksExactMut<'_, T>
```

Similarly, there are functions for reverse constant sized chunking
```rust,ignore
// For Some(&['e', 'm']), Some(&['o', 'r']), Some(&['l']) 
pub fn rchunks(&self, chunk_size: usize) -> RChunks<'_, T>

// For Some(&mut ['e', 'm']), Some(&mut ['o', 'r']), Some(&mut ['l']) 
pub fn rchunks_mut(&mut self, chunk_size: usize) -> RChunksMut<'_, T>

// For Some(&['e', 'm']), Some(&['o', 'r']), None 
pub fn rchunks_exact(&self, chunk_size: usize) -> RChunksExact<'_, T>

// For Some(&mut ['e', 'm']), Some(&mut ['o', 'r']), None 
pub fn rchunks_exact_mut(&mut self, chunk_size: usize) -> RChunksExactMut<'_, T>
```

#### Predicated chunking
Predicated chunking is when chunking is performed depending on a predicate. In general,
these functions expect a closure implementing `FnMut(...)` (yes, even non mutating `chunk_by`).
```rust,ignore
pub fn chunk_by<F>(&self, pred: F) -> ChunkBy<'_, T, F> ⓘ
where
    F: FnMut(&T, &T) -> bool,
```
> TODO: Figure out why is `FnMut` required here!

The following example stands:
```rust
let slice = &[1, 1, 1, 3, 3, 2, 2, 2];

let mut iter = slice.chunk_by(|a, b| a == b);

assert_eq!(iter.next(), Some(&[1, 1, 1][..]));
assert_eq!(iter.next(), Some(&[3, 3][..]));
assert_eq!(iter.next(), Some(&[2, 2, 2][..]));
assert_eq!(iter.next(), None);
```

### Repeat, Reverse, Join & Binary Search
