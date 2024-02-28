# Blink

## Description

```jsx
In the Blink of an eye, the reverse challenge unveiled secrets that should forever remain buried and unrecoverable.
```

## Solution
After downloading the challenge files, I see there are many files:

- 069b823.raw
- 31b3f8b.raw
- 4da6ff5.raw
- 8e1f2bc.raw
- acd3730.raw
- ae329eb.raw
- f488e59.raw
- blink

For the first step, I checked the blink file in the DIE (Detect It Easy) application to understand what type of executable it is:

![Untitled](Blink%20b008195dadce4ae3940e509ae8706479/Untitled.png)

As shown in the picture, It’s an ELF Linux executable. I opened it in IDA but after seeing the graph, I decided to check it later. At that moment I preferred to understand the program interactively. The program just returns what we write in stdin after a null terminator byte. I checked the other files but suddenly, I noticed that those are like ASCII arts and they complete each other. I decide to cat all of the files and pipe the output to the blink program. Its output is:

![Untitled](Blink%20b008195dadce4ae3940e509ae8706479/Untitled%201.png)

Then I copied the output to an editor and the result was:

![Untitled](Blink%20b008195dadce4ae3940e509ae8706479/Untitled%202.png)

```
____  _   _    _    ____  ____   ____  __  ___     _____  _ _   _     ____  _                 _     _____  _____   __  ___       ____       ____  _  _              ___  _ _       ___     _   _ _____ _____     _     ___   __  _____      _          ____        _  _       _____        ____      ___
/ ___|| \ | |  / \  |  _ \|  _ \ / /  \/  |/ _ \ __|___  |(_) \ | |   | ___|| |_ __ ___  _ __ | |   |___ / |_   _|__\ \/ / |_    / ___|  ___|  _ \| || |  _ __ ___  ( _ )| / |_ __ / _ \   | \ | | ____|___ /  __| |   / _ \ / _||___  | __ / \   _ __ | ___| _ __ | || |  _ _|___ / _ __  / ___|   _| \ \
\___ \|  \| | / _ \ | |_) | |_) | || |\/| | | | / __| / / | |  \| |   |___ \| | '_ ` _ \| '_ \| |     |_ \   | |/ _ \\  /| __|   \___ \ / __| |_) | || |_| '_ ` _ \ / _ \| | | '_ \ (_) |  |  \| |  _|   |_ \ / _` |  | | | | |_    / / '__/ _ \ | '_ \|___ \| '_ \| || |_| '__||_ \| '_ \| |  | | | | || |
 ___) | |\  |/ ___ \|  __/|  __< < | |  | | |_| \__ \/ /  | | |\  |    ___) |_| | | | | | |_) | |___ ___) |  | |  __//  \| |_     ___) | (__|  _ <|__   _| | | | | | (_) | | | | | \__, |  | |\  | |___ ___) | (_| |  | |_| |  _|  / /| | / ___ \| | | |___) | |_) |__   _| |  ___) | | | | |__| |_| |_| > >
|____/|_| \_/_/   \_\_|   |_|   | ||_|  |_|\___/|___/_/___|_|_| \_|___|____/(_)_| |_| |_| .__/|_____|____/___|_|\___/_/\_\\__|___|____/ \___|_| \_\  |_| |_| |_| |_|\___/|_|_|_| |_| /_/___|_| \_|_____|____/ \__,_|___\___/|_|___/_/ |_|/_/   \_\_| |_|____/| .__/   |_| |_| |____/|_| |_|\____\__, (_)| |
                                 \_\                 |_____|     |_____|                |_|             |_____|             |_____|                                                   |_____|                     |_____|    |_____|                         |_|                                |___/  /_/
```

---