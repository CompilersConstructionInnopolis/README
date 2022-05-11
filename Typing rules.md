(functype sum (int int int))
(func sum (a b) (plus a b)) - for a function we must declare functype firstly.
(type Time int) - a user can define aliases with a use of primitives.
(data (Either a b) (Left a) (Right b)) - this is the way a user defines own data types which can be parsed via pattern matching.
The return keyword inside a function must return the type defined in functype. It can’t return  if the function returns nothing.
The break keyword inside a function can be used only in such functions whose return type is nothing.
