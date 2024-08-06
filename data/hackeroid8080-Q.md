Description:
State variables slotIndexSelectionPoint and numberIndexSelectionPoint are not initialized, which might lead to unintended default values.

Proof of Concept:

uint256 public slotIndexSelectionPoint;
uint256 public numberIndexSelectionPoint;

Recommendation:
Initialize these variables in the constructor or provide default values during declaration.

uint256 public slotIndexSelectionPoint = someDefaultValue;
uint256 public numberIndexSelectionPoint = someDefaultValue;
