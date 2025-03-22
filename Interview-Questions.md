# Interview Questions

## Q. Why unmanaged code is called "unmanaged"?
Ans: 
- It is called unmanaged as it is not managed by .net CLR. Like CLR automatically manages the memory allocation & garbage collection in managed code but in unmanaged this is the responsibility of developer.
- Also other CLR features like type safety, exception handling have to be managed explicitly by developer.
- And unmanaged code runs directly on OS or hardware without protective features of CLR. 
Benefit of Unmanaged code:
- It allow more direct interaction with the hardware or other low-level operations as also give control over memory allocation & deallocation.
- It is more efficient in low-level operation as it bypass CLR management like garbage collection.

## Q. How managed code access unmanaged code?
Ans: There are few ways for managed code (code running under the .NET runtime, like C#) to interact with unmanaged code (code that runs directly on the Windows operating system, like native C or C++). Below is the most common way:
using DllImport: In below example, MessageBox() from user32.dll is called, which is an unmanaged function.

```cs
	[DllImport("user32.dll", CharSet = CharSet.Auto)]
    public static extern int MessageBox(int hWnd, string text, string caption, uint type);

    static void Main()
    {
        MessageBox(0, "Hello, World!", "Interop Example", 0);
    }
```
