# whook

WHOOK is a web hook for handling Tradingview Alerts to Kucoin and Bitget. Other exchanges may be added in the future.

Whook prioritizes realiability over speed. If you're looking for high frequency trading, this is not for you.
It will do everything it can to fullfill orders, including reducing the quantity or the order when the balance is not enough.


##### Disclaimer: This project is for my personal use. I'm not taking feature requests.

Currently supported exchanges:
- Kucoin futures
- Bitget futures
- Coinex futures

Hedge mode is not supported. I'm only using one side mode.


### ALERT SYNTAX ###

#### As plain text:

* Symbol format:: ETHUSDT, ETH/USDT, ETH/USDT:USDT

* Account id: Just add the id. No command associated. Account id must include at least one non-numeric character and obviously it shouldn't be the same as any of the command names.

* Commands:<br>
buy or long - places buy order.<br>
sell or short - places sell order.<br>
position or pos - goes to a position of the given value (no matter what the current position is).<br>
close - closes the position (position 0 also does it).<br>

* Quantities:<br>
[value] - quantity in contracts. No command associated, just the number.<br>
[value]$ - quantity in USDT. No command associated. Just the number and the dollar sign.<br>
[value]x or x[value] - defines the leverage.<br>

Examples:<br>
- Buy command using USDT:<br>
[account_id] [symbol] [command] [value in USDT] [leverage] - **kucoin000 ETH/USDT buy 300$ x3**<br>

- Position command using contracts:<br>
[symbol] [command] [value in contracts] [leverage] [account_id] - **ETH/USDT position -300 x3 kucoin000**<br>
Notice: This is a short position. For a long position use a positive value. Same goes when the value is in USDT<br>

- Close position<br>
[account_id] [symbol] [command] - **kucoin000 ETH/USD close**<br>

Several orders can be included in the same alert, separated by line breaks. For example, you can send the orders for 2 different accounts inside the same alert.

#### As JSON message:

JSON Messages are barely supported (I don't use them). Only accepts one alert per message and direct USDT orders aren't implemented.
Orders must come in contracts. The skeleton of the parser is there for anyone to complete it, but don't expect it to fully work as is.

{<br>
"symbol": "BTC/USDT",<br>
"command": "buy",<br>
"quantity": "12",<br>
"leverage": "3",<br>
"id" : "kucoin000"<br>
}

synonims: symbol, ticker // command, cmd, action



### API KEYS ###
When you first launch the script it will generate a json file. This file is a template to fill the accounts API data. This file can contain more than one account. It looks like this:


[<br>
&emsp;	{<br>
&emsp;&emsp;		"EXCHANGE":"kucoinfutures", <br>
&emsp;&emsp;		"ACCOUNT_ID":"your_account_name", <br>
&emsp;&emsp;		"API_KEY":"your_api_key", <br>
&emsp;&emsp;		"SECRET_KEY":"your_secret_key", <br>
&emsp;&emsp;		"PASSWORD":"your_API_password"<br>
&emsp;	}<br>
]<br>


You have to fill your API key and SECRET key information in the accounts.json file.<br>
The ACCOUNT_ID field is the name you give to the account. It's to be included in the alert message to identify the account.<br>
The EXCHANGE field is self explanatory. Valid exchange names are: "**kucoinfutures**", "**bitget**", "**bingx**", "**coinex**" and "**mexc**".<br>
The password field is required by Kucoin and Bitget but other exchanges may or may not use it. If your exchange doesn't give you a password when creating the API key just leave the field blank.<br>


###  HOW TO SET UP ###

If you want to go for a quick effortless test I recommend to copy/paste the script into a free account at 'https://replit.com'. It installs all modules for you so you don't have to do anything. Do **not** use it to host an actual server. It goes idle as soon as you close the browser, and anyone can see your API keys.

Local install for testing/working on the script:

If you have experience with python: It requires to pip install ccxt and flask.<br>
The following instructions are for Windows. If you are a Linux user I'm confident you know are familiar with the proccess.

- Install the latest version of Python from their website. During the installation make sure to *enable the system PATH option* and at the end of the installation *allow it to unlimit windows PATH length*

- Install Visual Code and in the extensions tab install the python extension. *Restart visual code*. In the terminal pip install ccxt and flask (just type 'pip install ccxt' and 'pip install flask'. Make sure you restarted VC after enabling the python extension. And make sure python was already installed)
https://code.visualstudio.com/download

With these you can already run the script, but it won't have access online. For giving it access to the internet you should use:

- ngrok. Create a free ngrok account. Download the last version of ngrok and unzip it. Launch the software and copy paste the auth code they give you on the website (with the authcode ngrok will be able to stay open forever). 
Then type in the ngrok console: "ngrok http 80". This will create an internet address that you can copy from the console. You have to add /whook to it to access the hook server.

Example of an address: https://e579-139-47-50-49.ngrok-free.app/whook

This address will continue stable until you close ngrok. Launching ngrok again will produce a new address.


### HOW TO HOST IN AWS ### 
(the easy way)

You can host a server in AWS EC2 for free. It can be a linux server or a windows server. You can find many tutorials in Youtube on how to do it.

I'm not a linux user so I struggled to open the ports in Linux. If you have experience in Linux this may be easy to you.

Here's a (slightly outdated) tutorial for windows: https://youtu.be/9z5YOXhxD9Q - I hosted it in a Windows_server 2022 edition which was the latest at the time I'm writing this readme. 

Basic steps are pretty similar to the local install:
- Download and install python. During the installation make sure to *enable the system PATH option* and at the end of the installation *allow it to unlimit windows PATH length*: https://www.python.org/downloads/
- Open the windows cmd prompt (type cmd in the windows search at the taskbar for the cmd prompt)
- pip install ccxt and pip install flask using the cmd prompt

With these you can already run the script, but it won't have access online. For giving it access to the internet you should use:

- ngrok. Create a free ngrok account. Download the last version of ngrok and unzip it. Launch the software and copy paste the auth code they give you on the website into the console (with the authcode ngrok will be able to stay open forever). Then type in the ngrok console: "ngrok http 80". This will create an internet address that you can copy. You have to add /whook to it to access the hook server.<br>

Example of an address: https://e579-139-47-50-49.ngrok-free.app/whook<br>

- You can launch the script by double clicking main.py (as long as you enabled the PATH options at installing python) or by creating a .bat file in the same directory as main.py like this:<br><br>

@echo off<br>
python.exe main.py<br>
pause<br>

If you have troubles with the cmd prompt or the bat file you can also install Visual Code in the server and run it from there.


### KNOWN BUGS ### 
- BingX contracSize and precision seem to be either wrong or work in a different scale than the rest of exchanges. The USDT to contracts conversion is returning wrong values. BingX support is uncomplete and I don't think I'll complete it. But I won't remove it either since most of it is implemented.
- Mexc has been in maintainance mode since 2022, and, while it connects and sets up fine, orders are denied. I think Mexc would be supported if the orders went thought, but I don't know if they will ever enable them again.


