# Visits

## Description

```
The origin of certain Windows binaries might not be Micro$oft Windows, posing a challenge when it comes to debugging them, like Vitis.
```

## Solution

After downloading the attachments, I checked the program in DIE (Detect it Easy) to have a better understanding of the executable type:

![Untitled](Visits%20a3941f36f74445f2b145470aeeca227e/Untitled.png)

It was an unpacked dotnet file, so I opened it in the dnspy and the flag is exactly in front of my eyes:)))

![Untitled](Visits%20a3941f36f74445f2b145470aeeca227e/Untitled%201.png)

---