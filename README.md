# Javascript for Automation
First off, I understand how difficult it is to use Apple's Javascript for Automation (JXA). JXA should be a great language for quickly automating stuff with tons of support, since it's based on javascript. However, it is not. Apple seems to have sidelined it, with is a real pity IMHO.

That being said, from what I've read, the few developers who use it, expect it to be supported for years to come -- and some of them are hoping for a revival. I'm with the first camp and would LOVE to see the latter.

Anyways, the following is a collection of JXA scripts that I developed for Apple Mail. Hope this helps someone going down this rabbit trail of zero documentation madness.

## Apple Mail
> _The following examples were built for ProtonMail. To adapt them for other clients, try changing the baseFolder from "Folders" to "Inbox". **Warning: This has NOT been tested yet.**_

### Get account properties
```js
var mail = Application('Mail'),
	account = mail.accounts['ProtonMail']

console.log(JSON.stringify(account.properties(), null, 4))
```

### Search for Messages
Search your inbox within your account for messages between two specific dates from a specific sender

```js
var mail = new Application("Mail"),
	account = mail.accounts['ProtonMail']

mail.accounts().forEach(account => {	
	var account = mail.accounts.whose({ name: { _contains: account.name() } }, {ignoring: 'case'}),
		mailbox = account.mailboxes.whose({ name: { _contains: 'INBOX' } }, {ignoring: 'case'})

	var messages = mailbox.messages.whose({ 
		sender: { _contains: 'email@example.com' },
		_and: [
			{ dateSent: { '>': new Date(2016,1,1) } },
			{ dateSent: { '<': new Date(2022,7,1) } },
		]
	});

	var subjects = []
	
	for (var i in messages) {
		subjects.push(messages[i].subject());
	}
		
	console.log(subjects);
});
```

### Find Messages from last 30 days
```js
var mail = new Application("Mail")

mail.accounts().forEach(account => {	
	var account = mail.accounts.whose({ name: { _contains: account.name() } }, {ignoring: 'case'}),
		mailbox = account.mailboxes.whose({ name: { _contains: 'INBOX' } }, {ignoring: 'case'}),
		messages = mailbox.messages.whose({ 
			dateSent: { '>': new Date(new Date().setDate(new Date().getDate() - 30)) }
		});
	
	for (var i in messages) {
		console.log(messages[i].subject())
	}
});
```

Flip the > sign, if you want to see messages older than 30 days...

### Find Unread messages older than 30 days old
This example includes an account selector

```js
var mail = new Application("Mail"),
	app = Application.currentApplication()

app.includeStandardAdditions = true

var accountList = mail.accounts().reduce((arr, x) => {
	arr.push(x.name()); return arr
}, [])

var accountName = app.chooseFromList(accountList, {
	withPrompt: "Select account to search:",
	defaultItems: accountList[0]
})[0]

var account = mail.accounts.whose({ name: { _contains: accountName } }, {ignoring: 'case'}),
	mailbox = account.mailboxes.whose({ name: { _contains: 'INBOX' } }, {ignoring: 'case'}),
	messages = mailbox.messages.whose({
		readStatus: false,
		dateReceived: { '<': new Date(new Date().setDate(new Date().getDate() - 30)) }
	});

for (var i in messages) {
	console.log(messages[i].subject())
}
```

### List mailboxes?

#### Option A
```js
var mail = Application('Mail')
var account = mail.accounts['ProtonMail']

var thisList = []

for (var i in account.mailboxes) {	
	if (account.mailboxes[i].container().properties().pcls == "container") {
		thisList.push(account.mailboxes[i].properties())
	}
}

console.log(JSON.stringify(thisList, null, 4))
```

#### Option B
Btw -- the variable for this doesn't make sense : P

```js
var accountList = account.mailboxes().reduce((arr, x) => arr + "," + x.name(), '').split(',').filter(n => n);
```

### Create new mailbox
```js
var mail = Application('Mail'),
	account = mail.accounts['ProtonMail'];

mb = mail.Mailbox({name: "Folders/Lorem D"});
account.mailboxes.push(mb);
```

### Create New Mailboxes
This example includes an account selector

```js
var baseFolder = "Folders/",
	mailboxNames = ["Lorem A", "Lorem B", "Lorem D"];

var mail = new Application("Mail"),
	app = Application.currentApplication();

app.includeStandardAdditions = true;

var accountList = mail.accounts().reduce((arr, x) => {
		arr.push(x.name()); return arr;
	}, []),
	accountName = app.chooseFromList(accountList, {
		withPrompt: "Select account to search:",
		defaultItems: accountList[0]
	})[0],
	account = mail.accounts[accountName];

for (mailboxName of mailboxNames) {
	var thisMailbox = baseFolder + mailboxName,
		mailbox = account.mailboxes.whose({ name: { _contains: mailboxName } });
		
	if (! mailbox[0].exists()) {
		mb = mail.Mailbox({name: thisMailbox});
		account.mailboxes.push(mb);
	}	
}
```

