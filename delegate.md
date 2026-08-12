
delegate is a type safe function pointer

delegate points to fxn and when we invole delegate , it invokes the function

``

```c#

accessmodifier  delegate returntype  delegatename(parameters);

public delegate void printsomething(string str);

```


this delegate can now point to any function with same signature

now we need to make this delegate to point to function , we need to create an instance for delegate

delegatename delegateinstance = new delegatename(functionname);

