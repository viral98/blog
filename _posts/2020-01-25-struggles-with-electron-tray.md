---
layout: post
title: Electron - Getting the tray right
---

Electron has been coming up as the choice of frame work to be used when porting a project which was formerly a web app to a desktop app (or adding additional functionalities when using the web app over on desktop - the case we handled at P2P.inc)

## Problems with the default setup

![image](https://user-images.githubusercontent.com/25403969/105564595-ec64f580-5d48-11eb-81a0-e425d9e62fe4.png)

The tray in electron serves as a quick access to the application's functionalities without necessarily fiddling around with the application window itself.

However, the actual usage goes much deeper than the above mentioned definition and is often overlooked until teams start running into major issues because of not configuring their tray correctly!

### Non functional keyboard shortcuts - Mac

You might be surprised to know that unless you configure your tray code correctly, the general accelerators like CMD+C CMD+V to certain mac specific accelerators like CMD+Q and CMD+W won't work at all.

It's funny to think, but we need to explicity define these inorder to make sure simple events like copy paste work fine

```javascript
 const menu = new Menu();
    menu.append(
      new MenuItem({
        label: "Application",
        submenu: [
          {
            role: "reload",
            accelerator: "F5",
            visible: false,
          },
          {
            role: "toggleDevTools",
            accelerator: "F12",
            visible: false,
          },
          { type: "separator" },
          {
            role: "close",
            accelerator: "CommandOrControl+W",
          },
          {
            role: "quit",
            accelerator: "CommandOrControl+Q",
          },
        ],
      })
    );
    menu.append(
      new MenuItem({
        label: "Edit",
        submenu: [
          {
            role: "undo",
            accelerator: "CommandOrControl+z",
          },
          {
            role: "redo",
            accelerator:
              process.platform === "darwin" ? "Command+Shift+z" : "Control+y",
          },
          { type: "separator" },
          {
            role: "cut",
            accelerator: "CommandOrControl+X",
          },
          {
            role: "copy",
            accelerator: "CommandOrControl+C",
          },
          {
            role: "paste",
            accelerator: "CommandOrControl+V",
          },
          { type: "separator" },
          {
            role: "selectAll",
            accelerator: "CommandOrControl+A",
          },
        ],
      })
    );

    Menu.setApplicationMenu(menu);
```

### Things that show up on mac will not show up on windows

While this is a no brainer at the first look, there are many things which will not show up the same way they do on mac vs on windows! This further emphasizes the fact that development teams need to ensure that they use a variety OS's while developing / performing QA even if electron has a general perception of being `set-it-up once and run it on any OS`.
We ran into this while adding a visual view to the working timer that employee's run while working - this timer gets displayed and refreshed by the tray's title - an API only available on mac!
![image](https://user-images.githubusercontent.com/25403969/105565039-d48e7100-5d4a-11eb-8669-d42c6bb4064b.png)

These were the things I came across while developing a cross-platform app using electron! Hope this proves helpful to your team !
