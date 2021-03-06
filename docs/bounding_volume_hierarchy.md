# `ArborX::BVH`
Defined in header [`<ArborX_LinearBVH.hpp>`](https://github.com/arborx/ArborX/blob/master/src/ArborX_LinearBVH.hpp)
```C++
template <typename MemorySpace>
class BoundingVolumeHierarchy;
```

The class template `ArborX::BoundingVolumeHierarchy` is a tree data structure that can be used to accelerate the search for geometrical objects in space.

ArborX defines the `ArborX::BVH` alias template for convenience.
```C++
template <typename MemorySpace>
using BVH = BoundingVolumeHierarchy<MemorySpace>;
```

## Template parameter
`MemorySpace`
: A valid Kokkos memory space.

## Member types
Member type | Definition
--- | ---
`memory_space` | `MemorySpace`
`size_type` | `MemorySpace::size_type`
`bounding_volume_type` | `ArborX::Box`

## Member functions
[`(constructor)`](#arborxbvhmemoryspacebvh)
: constructs the tree data structure.

[`size`](#arborxbvhmemoryspacesize)
: returns the number of leaves stored in the tree.

[`empty`](#arborxbvhmemoryspaceempty)
: checks whether the tree has no leaves.

[`bounds`](#arborxbvhmemoryspacebounds)
: returns a bounding volume able to contain all leaves stored in the tree.

[`query`](#arborxbvhmemoryspacequery)
: finds all leaves that satisfy given predicates, e.g. nearest to a point or intersecting with a box or a sphere.

# `ArborX::BVH<MemorySpace>::BVH`
```C++
BVH() noexcept; // (1)

template <typename ExecutionSpace, typename Primitives>
BVH(ExecutionSpace const& space, Primitives const& primitives); // (2)
```
1) Default constructor.  Constructs an empty tree.  
2) Constructs a bounding volume hierarchy from the given data source.
## Parameters
`space`
: the execution space.  
`primitives`
: geometrical objects one wishes to index.
#### Type requirements
A specialization of [`ArborX::Traits::Access`](https://github.com/arborx/ArborX/blob/master/docs/traits.md#arborxtraitsaccess) must match `Primitives` as the first template argument and `PrimitivesTag` as second argument.  
The return type of `ArborX::Traits::Access<Primitives,PrimitivesTag>::get()` must decay either to `ArborX::Point` or `ArborX::Box`.
ArborX provides specializations for Kokkos views but a user may specialize it for their types.

## Complexity
$O(N log(N))$, where `N = ArborX::Traits::Access<Primitives,PrimitivesTag>::size(primitives)` is the number of primitives passed to the constructor.
## Exceptions
Memory allocation with Kokkos may throw.
## Notes

## Example
```C++
#include <ArborX_LinearBVH.hpp>

#include <Kokkos_Core.hpp>

#include <iostream>

std::ostream &operator<<(std::ostream &os, ArborX::Point const &p)
{
  os << '(' << p[0] << ',' << p[1] << ',' << p[2] << ')';
  return os;
}

int main(int argc, char *argv[])
{
  Kokkos::ScopeGuard guard(argc, argv);

  Kokkos::View<ArborX::Point *> cloud("point_cloud", 1000);
  Kokkos::parallel_for(1000, KOKKOS_LAMBDA(int i) {
    cloud[i] = {{(float)i, (float)i, (float)i}};
  });

  using memory_space = decltype(cloud)::memory_space; // where to store the tree
  using execution_space = decltype(cloud)::execution_space; // where to execute code
  ArborX::BVH<memory_space> bvh{execution_space{}, cloud};

  auto const box = bvh.bounds();
  std::cout << box.minCorner() << " - " << box.maxCorner() << '\n';

  return 0;
}
```
Output
```
(0,0,0) - (999,999,999)
```
## See also
[`query`](#arborxbvhmemoryspacequery)
: search for all primitives that meet some predicates.  
[`bounds`](#arborxbvhmemoryspacebounds)
: returns the bounding volume that contains all leaves.

# `ArborX::BVH<MemorySpace>::size()`
```C++
size_type size() const noexcept;
```
Returns the number of leaves in the tree.
## Parameters
(none)
## Return value
The number of leaves in the tree.
## Complexity
Constant.
## See also
[`empty`](#arborxbvhmemoryspaceempty)
: checks whether the tree is empty.  
[`bounds`](#arborxbvhmemoryspacebounds)
: returns the bounding volume that contains all leaves.

# `ArborX::BVH<MemorySpace>::empty()`
```C++
bool empty() const noexcept;
```
Checks if the tree has no leaves.
## Parameters
(none)
## Return value
`true` if the tree is empty, `false` otherwise.
## Complexity
Constant.
## See also
[`size`](#arborxbvhmemoryspacesize)
: returns the number of leaves.

# `ArborX::BVH<MemorySpace>::bounds()`
```C++
bounding_volume_type bounds() const noexcept;
```
Returns the smallest axis-aligned box able to contain all leaves stored in the tree.
## Parameters
(none)
## Return value
The smallest bounding volume that contains all leaves stored in the tree or an invalid bounding volume if the tree is empty.
## Complexity
Constant.
## See also

# `ArborX::BVH<MemorySpace>::query()`
```C++
template <typename ExecutionSpace, typename Predicates>
void query(ExecutionSpace const& space, Predicates const& predicates, ...) const;
```

## Parameters
`space`
: Execution space that specifies where to execute code.  
`predicates`
: Predicates to check against the primitives.

#### Type requirements
A specialization of [`ArborX::Traits::Access`](https://github.com/arborx/ArborX/blob/master/docs/traits.md#arborxtraitsaccess) must match the `Predicates` as first template argument and `PredicatesTag` as second argument.  
The static member function `ArborX::Traits::Access<Predicates,PredicatesTag>::get()` return type must decay to a valid ArborX predicate.  
Such predicate may be generated by one of the functions listed below:
* [`ArborX::nearest()`](https://github.com/arborx/ArborX/blob/master/docs/predicates.md#arborxnearest)
* [`ArborX::intersects()`](https://github.com/arborx/ArborX/blob/master/docs/predicates.md#arborxintersects)

`indices`
: position of the primitives that satisfy the predicates.  
`offsets`
: predicate offsets in indices.

`indices` stores the indices of the objects that satisfy the predicates.  `offset` stores the locations in the `indices` view that start a predicate, that is, `predicates(q)` is satisfied by `indices(o)` for `primitives(q) <= o < primitives(q+1)`.
Following the usual convention, `offset(n) = nnz`, where `n` is the number of queries that were performed and `nnz` is the total number of collisions.
## Return value
(none)
## Complexity
$O(M log(N)$ where `M = ArborX::Traits::Access<Predicates,PredicatesTag>::size(predicates)` and `N = this->size()`.
## Exceptions
Memory allocation with Kokkos may throw.
## Notes
## Example
## See also
