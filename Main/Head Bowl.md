A [[Machine Learning]] Library.
---



Naming standards
---
Each component, Layer, Net, or Loss functions,  etc. has a generic overload. The generic alternative has the same name as  it's numeric-type-specific counterpart. 

Numeric-type-specific are named with Hungarian notation. 
Example:
	`[Generic type] Net` -> `[Half targeting] Net16`
	`[Generic type] Net` -> `[Float targeting] Net32`
	`[Generic type] Net` -> `[Double targeting] Net64`
	`[Generic type] Net` -> `[Decimal targeting] Net128`



TODO
---
Test performance differences for different implementations of the generic classes.
