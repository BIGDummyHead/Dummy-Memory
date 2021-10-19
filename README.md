# Dummy Memory

#### This library has simple methods to read and write memory, relies heavily on IntPtr, and has a unique Detouring system using Code Caves.

This package is available on Nuget for x86 and x64 archetypes. [DM.dll](https://www.nuget.org/packages?q=DM-) on Nuget. This package now works on Nuget, it did not have the correct file structure.

### Copy Me 

```csharp

using DummyMemory;

class Program
{
    static Memory mem = new Memory("ac_client");
    static IntPtr PlayerBase => mem.Base + 0x109B74;
    
    static void Main()
    {
       int offset = 0xF8;
       
       //can return default 
       int health = mem.Read<int>(PlayerBase, offset);
       
       bool didWrite = mem.Write<int>(PlayerBase, 100 + health, offset);
    }
    
    static Vector3 GetPlayerPos()
    {
        int offset = 34;
        
        //we can even use the Write<T> to do the same thing
        //mem.Write<Vector3>(PlayerBase, vec3, offset);
        return mem.Read<Vector3>(PlayerBase, offset);
    }
    
    public Vector3
    {
       public float x, z, y;
    }
}
``` 

## Creating Detours

Detours is coined as AoB injection by Cheat Engine, it allows us to essentialy replace the game's code by jumping to a allocated region of memory, and then returning to the next address we just wrote our Jump code to. I've tried to make code caves as friendly as possible by simply supplying basic info about your AoB injection and then being able to say `cave.Inject();` or `cave.Eject();`

Below you can see how to write your own code cave! 

```csharp

using DummyMemory;
using DummyMemory.Detouring;

//this code cave will simply increase instead of decrease the ammo count.
static Memory mem = new Memory("ac_client");
static void Main()
{
    //aob pattern to find
    string aob = "FF 0E 57 8B 7C 24 14";
    
    //the bytes we are going to write to our allocated block of memory
    //optionally we could use a string formatted version of this:
    //"FF 06 57 8B 7C 24 14";
    byte[] inject =
    {
        0xFF, 0x06, 0x57, 0x8B, 0x7C, 0x24, 0x14
    };

    //create a new instance of cave, make sure that the aob has not been changed or written to by another 'Cave' or CheatEngine
    
    Cave cave = new Cave(mem, aob, inject, new Cave.Allocation
    {
        //size of memory block
        memorySize = 1000,
        //replacement size, jmp + nops
        replacementSize = 7
    });
   
    //inject our cave, bytes from both the address and the region our inserted
    cave.Inject();

    Console.ReadLine();

    //our region of memory is deallocated and our original bytes are written back to the address
    cave.Eject();
}
```

## Injecting DLLs

Injecting DLLs can sometimes be frustrating in C# so I've made it pretty simple in this latest update. But you'll have to run your Program in ADMIN mode to inject them.
Let's take a look at some of the results you may get when injecting your own dll.

* Dll Does Not Exist - The dll you requested to inject does not exist
* Not Admin - Your program is not in Administrator mode
* Bad Pointer - One of the pointers when injecting your Dll was equal to 0
* Injected - No errors happened!
* Close Fail Injected - Handle(s) from your injection were not properly closed

So how do we do it?

We can either do it throught a static method or through an instance of Memory.cs as such

```csharp

Memory m = new Memory("ac_client");
//there you go you sucessfully injected your dll
Memory.InjectionStatus status = m.Inject("C:\\Path\\target.dll");

//or
Memory.InjectionStatus status = Memory.Inject(Process.GetProcessesByName("ac_client")[0], "C:\\Path\\target.dll");

```