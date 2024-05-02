# This custom build of KCommando has been adapted for TeirTests. 

## Features
All these features have a modular structure and you can edit all of these modules and integrate them for your projects.
1. [Integrations](#kcommando-integrations)
2. [Creating Slash Command](#how-to-create-a-slash-command)
3. [Creating Subcommands](#how-to-create-a-subcommand)
4. [Handling Buttons](#how-to-handle-buttons)
5. [Parameterized Constructors](#how-to-use-parameterized-constructor)

# KCommando Integrations

### Integration Usage For JDA
```java
package com.example.mybot;

public class Main {

    public void main(String[] args) throws Exception {
        JDA jda = JDABuilder.createDefault("TOKEN").build();
        jda.awaitReady();
        
        JDAIntegration integration = new JDAIntegration(jda);
        
        KCommando kcommando = new KCommando(integration)
              .setOwners(00000000L, 00000000L) // varargs LONG
              .addPackage("com.example.mybot") // package to analyze
              .setCooldown(5000L) // 5 seconds as 5000 ms
              .setPrefix("!")
              .setReadBotMessages(false) // default false
              .setUseCaseSensitivity(false) // default false
              .setAllowSpacesInPrefix(true) // default false
              .setDefaultFalseMethodName("defaultCallback")
              .setVerbose(true) // for logging
              .build();
    }
}
```

That's it. Now, we can create slash commands or classic commands.

### How To Create A Slash Command
You don't have to identify the `guildId`. If you don't, it will be a global command. Also options are optional. It's okay if the method is static. 

```java
package com.example.mybot.slash;

public class MySlashCommands {
    
    @HandleSlash(name = "hello", desc = "Test command.", guildId = 000000L,
                 options = @Option(type = OptionType.STRING, name = "yourName", required = true))
    public void helloCommand(SlashCommandInteractionEvent e) {
		
        e.deferReply(false).queue();
        String name = e.getOption("yourName").getAsString();
        e.getHook().sendMessage("Hello " + name + "!").queue();
        
    }
    
    @HandleSlash(name = "ping", desc = "Pong!")
    public static void pingCommand(SlashCommandInteractionEvent e) {
        
        e.reply("Pong!!").addActionRow(
                Button.primary("buttonHello", "Button Text"),
                Button.secondary("processData", "Process")
        ).queue();
        
    }
}
```

### How To Create A Subcommand
To create a subcommand, you have to register the group on the JDAIntegration object. Here is an example:
```java
package com.example.mybot;

public class Main {

    public void main(String[] args) throws Exception {
        JDA jda = JDABuilder.createDefault("TOKEN").build();
        jda.awaitReady();
        
        JDAIntegration integration = new JDAIntegration(jda);
	integration.addSubCommandGroup(new CommandDataImpl("manage", "Manage settings"));
        
        KCommando kcommando = new KCommando(integration)
              .setOwners(00000000L, 00000000L) // varargs LONG
              .addPackage("com.example.mybot") // package to analyze
              .setCooldown(5000L) // 5 seconds as 5000 ms
              .setPrefix("!")
              .setReadBotMessages(false) // default false
              .setUseCaseSensitivity(false) // default false
              .setAllowSpacesInPrefix(true) // default false
              .setDefaultFalseMethodName("defaultCallback")
              .setVerbose(true) // for logging
              .build();
    }
}
```

And heres how to create the subcommand itself:
```java
package com.example.mybot.slash;

public class MySlashCommands {
    
    @HandleSlash(name = "hello", desc = "Test command.", guildId = 000000L, subCommand = true, parentGroup = "manage",
                 options = @Option(type = OptionType.STRING, name = "yourName", required = true))
    public void helloCommand(SlashCommandInteractionEvent e) {
		
        e.deferReply(false).queue();
        String name = e.getOption("yourName").getAsString();
        e.getHook().sendMessage("Hello " + name + "!").queue();
        
    }
}
```

Things to note:
- Make sure the names are lower case in the registration of the subcommand group
- Make sure that the group name is registered (it will throw an error)

### How To Handle Buttons
```java
package com.example.mybot.buttons;

public class Hello {
    
    @HandleButton("buttonHello")
    public void helloButton(ButtonInteractionEvent e) {
        // ...
    }
    
    @HandleButton("processData")
    public void processor(ButtonInteractionEvent e) {
        // ...
    }
    
}
```

## How To Use Parameterized Constructor
```java
package com.example.mybot;

public class Example {
    
    // kcommando doesn't have a database manager
    // this is example for how to use parameterized classes with kcommando
    private final DatabaseManager databaseManager;
    
    public Example(DatabaseManager databaseManager) {
        this.databaseManager = databaseManager;
    }

    @HandleCommand(name = "ListDatabase", aliases = {"db", "listdb"})
    public void command(MessageReceivedEvent e) {
        String example = databaseManager.query("SELECT * FROM logs");
        // ...
    } 
}
```

```java
package com.example.mybot;

public class Main {
    public void main(String[] args) throws Exception {
        JDA jda = JDABuilder.createDefault("TOKEN").build();
        jda.awaitReady();
        
        JDAIntegration integration = new JDAIntegration(jda);
        KCommando kcommando = new KCommando(integration)
                .setPackage("com.example.mybot") // package to analyze
                .setPrefix("!")
                .setVerbose(true)
                .build();

        DatabaseManager databaseManager = new DatabaseManager();
        
        // this class includes command
        // kcommando will use this instance while executing the command
        Example myObject = new Example(databaseManager);

        // also you can do this before build the kcommando
        kcommando.registerObject(myObject); // <--------
    }
}
```
