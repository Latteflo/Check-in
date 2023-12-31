# MyBeCode Check-in Automation

---

# 0. What is this all about?

## 0.1. **The problem** 🧐

It’s easy to forget to click on the buttons, or to do it one minute late.

## 0.2. The goal 🎯

Let’s give Diogo some rest and consistently click on the button... but without thinking about it.

## 0.3. The path 🛤️

Since the first week, I’ve been trying to automate the check-in on my.becode when I had some time.

After a first try with JS only, which worked but was not handy, I aimed at automating it to the fullest: we wouldn’t need to bother to even think about opening the page.

And after quite a few attempts and various paths taken, here it is. Enjoy. 😄

!https://c.tenor.com/yEEJWgag3_cAAAAd/jim-carrey-its-a-live.gif

---

# 1. Playwright 🎭

Install Playwright inside the directory (let’s say `~/code/mybecode-chekin/`). You’ll need nodejs and npm.

```bash
npm i -D playwright
```

*Playwright is an open-source automation tool backed-up by Microsoft. More info on their website:* 

[Fast and reliable end-to-end testing for modern web apps | Playwright](https://playwright.dev/)

<aside>
ℹ️ If you want to try it a bit before running a script with it, I suggest you do so by typing in your terminal:
`npx playwright codegen https://www.wikipedia.org` 
This will open a Chromium webpage on Wikipedia. Start clicking on a few links.
A secondary window traces what you’re doing, where you’re clicking, etc. You can then copy this code to run your own script.
The code is generated for JavaScript, Python, Java and C#. (See the ‘target’ in the upper-right corner.)

</aside>

---

# 2. Create two JavaScript scripts

## 2.1. What’s the code?

You’ll need 2 js scripts. One for home, one for BeCode, only differentiated by one letter (-h / -b).

I suggest `mybecode-checkin-h.js`

(Honestly, one file with a variable passed down from the bash script (cf. next step) should be enough, but I couldn’t make it work, so I resorted to using two.)

- **Here it is** ⤵️
    
    ```jsx
    const { firefox } = require('playwright');
    
    (async () => {
        const browser = await firefox.launch({
            locale: 'fr-BE',
            timezone: 'Europe/Brussels'
        });
    
        const context = await browser.newContext();
        
        await context.addCookies([{
            name: 'user_session',
            value: '******',
            domain: 'github.com',
            path: '/',
            expires: -1,
            httpOnly: true,
            secure: true,
            sameSite: 'None',
          }]);
        
        const page = await context.newPage();
    
        // Go to https://my.becode.org/
        await page.goto('https://my.becode.org/');
        // Click text=Login via GitHub
        const [page1] = await Promise.all([
            page.waitForEvent('popup'),
            page.click('text=Login via GitHub')
        ]);
    
        // Close page
        await page1.close();
    
        // Go to https://my.becode.org/dashboard
        await page.goto('https://my.becode.org/dashboard');
    
        if (page.$('text=Login via Github') !== null) {
            await page.click('text=Login via GitHub');
        }
    
        // Click on discard
        await page.click('text=discard');
    
        // Click on BeCode
        // await page.click('text=@ BeCode');
    
        // Click on Home
    		await page.click('text=@ Home');
        
    		let time = new Date();
    		// 1st button, between 8:45 and 9:00
    		if (time.getHours() === 8 && time.getMinutes() >= 50) {
            // Click on button 09:00
            await page.click('text=09:00');
    		}
    		// 2nd button, at 12:30 or after
    		else if (time.getHours() === 12 && time.getMinutes() >= 30) {
            // Click on button 12:30
            await page.click('text=12:30');
    		} 
    		// 3rd button, between 13:20 and 13:30
    		else if (time.getHours() === 13 && time.getMinutes() >= 24) {
            // Click on button 13:30
            await page.click('text=13:30');
    		} 
    		// 4th button, at 17:00 or after
    		else if (time.getHours() === 17 && time.getMinutes() >= 0) {
            // Click on button 17:00
            await page.click('text=17:00');
    		}
        
        await page.close();
        await context.close();
        await browser.close();
    })();
    ```
    

Then, create a second js script called `mybecode-checkin-b.js`. Uncomment the line to click on `BeCode`, and comment the line to click on the `Home` button.

<aside>
ℹ️ *You might have seen that we’re using Firefox: it’s the only browser that isn’t flagged as a bot by my.becode.
But don’t worry if you don’t have it, Playwright downloaded it for you. (By default, Playwright installs Firefox, Chromium and Webkit.)*

![ *This is what you see when trying to connect through Chromium* ⤴️
 *Fat chance.* ¯\_ (ツ)_/¯ ](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/51ab72d1-66dd-4e3c-8c2d-1f8f4fdd7372/Untitled.png)

 *This is what you see when trying to connect through Chromium* ⤴️
 *Fat chance.* ¯\_ (ツ)_/¯ 

</aside>

## 2.2. Make it your own

I lied to you: that code won’t work. Well, I didn’t *really* lie. But you need to adapt it quite a bit to your own situation.

From line 11 to line 20, you’ll find the cookie GitHub sets for you to log in. We need it in order to bypass the connection process, which is a dead-end if you activated the two-factor authentication on GitHub.

In short, you will have to find your own GitHub cookie and place it:

- Line 13: your user session ID

<aside>
ℹ️ **Finding your cookies on Firefox:**
*For Firefox, the easiest way to retrieve your cookies is to go on GitHub, open your developer’s tools, and go on the tab “Storage”.*

Note: I don’t know if you can feed it Chrome cookies. You can try it if you feel like it: go into *`Settings` > `Privacy` > `Cookies & other data` > `Show all the cookies and data`.* (If you do, tell me if it worked...)

</aside>

---

# 3. Create a .sh file

## 3.1. Let’s code

That bash file (let’s call it `mybecode-checkin.sh`) will contain:

```bash
#! /usr/bin/bash

export DISPLAY=:0

workingPlace=$(cat ~/code/mybecode-checkin/working-place)

# if there's nothing in the file, choose the default
if [[ $workingPlace != "b" && $workingPlace != "h" ]] ; then
	day=`date +%u`

	# Monday or Tuesday: BeCode
	if [[ $day -eq 1 || $day -eq 2 ]] ; then
		workingPlace="b"
	# the rest of the time: home
	else
		workingPlace="h"
	fi
fi

node /home/$USER/code/mybecode-checkin/mybecode-checkin-${workingPlace}.js
```

<aside>
ℹ️ **Notes**:

1. `#! /usr/bin/bash` is called a *shebang* (if you want to look it up). It gives the path to the bash tool and is needed for the script to run properly. 
    
    *The path might be different on your computer. Type `which bash` in your terminal to locate it.*
    
    *You may also use zsh instead of bash. Type `which zsh` to locate it.*
    
2. Adapt lines **5** and **20** to your folder path.
3. Line 20 is the path to your .js file. The path should be absolute (i.e. starting at the root) because of the next step.
4. For it to work (cf. next step), you need to give yourself the execution rights on the file: type `chmod 764 /path/to/your/file.sh` in the terminal.
</aside>

<aside>
🧠 **Explanations:**

The script first checks whether there is anything in the file called “working-place”. See step 5 to understand why it’s there. It copies the value of the file inside a variable, `workingPlace`.

Then, it checks a few conditions. If `workingPlace` is different from “b” and “h” (*i.e.:* *most probably when it’s empty*), it checks the day of the week (`date +%u`). If it’s a Monday or a Tuesday, it sets `workingPlace` to “b”. Else, it sets it to “h”.

Finally, it runs the corresponding js file through Nodejs.

</aside>

---

# 4. Add it to your computer’s routine: cron

## 4.1. What’s Cron?

> **Cron** is a program used to automatically execute scripts, commands or softwares at a specified date and time, or according to a cycle defined in advance.
Each user has a **crontab** file (for *cron table*) that allows them to fill in the actions to execute.
Cron is sometimes called “task scheduler” or “scheduled tasks manager”.
> 

*More info: https://www.youtube.com/watch?v=QZJ1drMQz1A*

## 4.2. How to set it up

*Alternatives for: (didn’t try them myself, obviously)*

- *MacOS:* https://alvinalexander.com/mac-os-x/launchd-plist-examples-startinterval-startcalendarinterval/
- *Windows:* https://stackoverflow.com/questions/7195503/setting-up-a-cron-job-in-windows **

In **Linux:**

In the terminal, type:

```bash
crontab -e
```

Select your editor, then copy:

```bash
# ┌———————————————— minute (0 - 59)
# | ┌———————————————— hour (0 - 23)
# | | ┌———————————————— day of month (1 - 31)
# | | | ┌———————————————— month (1 - 12)
# | | | | ┌———————————————— day of week (0 - 6) (Sunday to Saturday)
# | | | | |
# | | | | |
# | | | | |
# * * * * *  command_to_execute

55 8 * * 1-5 /home/$USER/code/mybecode-checkin/mybecode-checkin.sh
31 12 * * 1-5 /home/$USER/code/mybecode-checkin/mybecode-checkin.sh 
25 13 * * 1-5 /home/$USER/code/mybecode-checkin/mybecode-checkin.sh
1 17 * * 1-5 /home/$USER/code/mybecode-checkin/mybecode-checkin.sh
```

... and save.

<aside>
🧠 **Explanations:**

1. The 4 last lines mean that the bash script will run every Monday to Friday (1-5), at 08:55, 12:31, 13:25 and 17:01.
    
    *See this or the video linked above for a better understanding:* 
    
    [Crontab.guru - The cron schedule expression editor](https://crontab.guru/#55_8_*_*_1-5)
    
2. Obviously, you should adapt the paths to your directory and your file.
</aside>

Check if it’s been updated by typing:

```bash
crontab -l
```

If it doesn’t output what you just entered, either you didn’t save, or maybe your editor isn’t taken into account. Change your editor by typing `select-editor` and retry.

<aside>
⚠️ I already said it, but you need to give the execution rights on your script, otherwise the cron job won’t run.

</aside>

---

# 5. Not at the default working place? Change it!

The default working place is set to BeCode on Mondays and Tuesdays (*cf. bash script*), and at home the rest of the time.

Should you need to change the default settings for some reason, you can do it simply by typing one of the two following commands in your terminal (*from the directory that you have set in the bash script, line 5*):

```bash
# being at BeCode on another day than Mon/Tue:
echo "b" > working-place

## OR

# being at home on Mon/Tue:
echo "h" > working-place
```

As soon as you come back to the default working place, you’ll need to reset the file:

```bash
cat /dev/null > working-place
```

## 5.1. Bonus tip: make *aliases* to make it easier

Those 3 commands can be a pain to type. So you could create 3 aliases instead. You just have to edit your `~/.bashrc` or `~/.zshrc` file (depending on which shell you’re using), find where you can add some aliases, and add for instance:

```bash
alias becodeb='echo "b" > ~/code/mybecode-checkin/working-place'
alias becodeh='echo "h" > ~/code/mybecode-checkin/working-place'
alias becodenull='cat /dev/null > ~/code/mybecode-checkin/working-place'
```

************************************************(Of course, set the aliases you want. `bcb, bch, bc0` would work just as well. Just make sure you don’t overwrite existing commands by doing so.)*

---

# 6. Caveats

1. That `working-place` fiddling is probably not the best way to do that, as you need to remember to do it and then remember to switch it off. But it’s flexible enough. If you know how to improve that part of the process, I’m all ears!
2. The cron jobs will run every week. So for the 2 * 1-week break we have, you might need to switch them off. (Just edit the crontab and comment the lines with #.)
(*Another way of doing it would be directly through the bash file, to condition the whole process depending on the dates. I was too lazy to do it, but if you have a functional (or nearly functional) code for it, share it!)*

---

# 7. Disclaimer

Bugs, network errors, shit happens. I’m not responsible if your button isn’t clicked. 😬

Once it’s all set up, I would advise you to check myBeCode 1 or 2 minutes after the script has run. If it doesn’t work: there’s a problem somewhere. If it works: check again for the rest of the day. After that, it should be fine in general and you don’t have to worry about it (you still need to open your computer on time, obviously). 😉

Still, I would advise you to check it when you think about it during the day: if there has been a problem, better spot it the earliest possible.

!https://c.tenor.com/9ZQtlsEVALIAAAAC/woo-dance.gif

---

# Enjoy ! 😄