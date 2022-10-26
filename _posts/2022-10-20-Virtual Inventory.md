# Virtual Inventory
**Main Class**

```java
import org.bukkit.inventory.ItemStack;
import org.bukkit.plugin.java.JavaPlugin;

import java.util.HashMap;
import java.util.List;
import java.util.Map;

public class Main extends JavaPlugin {

    public static Map<String, ItemStack[]> menus = new HashMap<String, ItemStack[]>();

    @Override
    public void onEnable() {
        System.out.println("Plugin has been enabled!");
        getCommand("inventory").setExecutor(new command());
        getServer().getPluginManager().registerEvents(new inventory(), this);
        saveDefaultConfig();

        if (this.getConfig().contains("data")) {
            this.restoreInvs();
        }
    }

    @Override
    public void onDisable() {
        menus = inventory.item;
        System.out.println("Plugin has been disabled!");

        if (!menus.isEmpty()) {
            this.saveInvs();
        }
    }


    public void saveInvs() {
        for (Map.Entry<String, ItemStack[]> entry : menus.entrySet()) {
            this.getConfig().set("data." + entry.getKey(), entry.getValue());
        }
        this.saveConfig();
    }

    public void restoreInvs() {
        this.getConfig().getConfigurationSection("data").getKeys(false).forEach(key ->{
            ItemStack[] content = ((List<ItemStack>) this.getConfig().get("data." + key)).toArray(new ItemStack[0]);

            menus.put(key, content);
        });
        inventory.item = menus;
    }
}
```


**Command Class**

```java
import org.bukkit.Bukkit;
import org.bukkit.command.Command;
import org.bukkit.command.CommandExecutor;
import org.bukkit.command.CommandSender;
import org.bukkit.entity.Player;

public class command implements CommandExecutor {

    @Override
    public boolean onCommand(CommandSender commandSender, Command command, String s, String[] strings) {
        if (commandSender instanceof Player) {
            openInventory event = new openInventory((Player) commandSender);
            Bukkit.getPluginManager().callEvent(event);
            return true;
        } else {
            commandSender.sendMessage("This command is only allowed to the players!");
            return false;
        }
    }
}
```


**Inventory Class**

```java
import org.bukkit.Bukkit;
import org.bukkit.Material;
import org.bukkit.entity.Player;
import org.bukkit.event.Event;
import org.bukkit.event.EventHandler;
import org.bukkit.event.HandlerList;
import org.bukkit.event.Listener;
import org.bukkit.event.inventory.InventoryCloseEvent;
import org.bukkit.event.inventory.InventoryOpenEvent;
import org.bukkit.inventory.Inventory;
import org.bukkit.inventory.ItemStack;


import java.util.HashMap;
import java.util.Map;

public class inventory implements Listener {
    public final Inventory inv;
    static Map<String, ItemStack[]> item = new HashMap<String, ItemStack[]>();

    public inventory() {
        inv = Bukkit.createInventory(null, 36, "Virtual Inventory");
    }



    @EventHandler
    public void openInventory(openInventory e) {
        e.player.openInventory(inv);
    }

    @EventHandler
    public void InventoryOpenEvent(InventoryOpenEvent e) {
        Player p = (Player) e.getPlayer();
        if (e.getInventory() == inv) {
            ItemStack air = new ItemStack(Material.AIR, 1);
            for (int i = 0; i <= 35; i++) {
                e.getInventory().setItem(i, air);
            }
            if (item.containsKey(p.getUniqueId().toString())) {
                int num = 0;
                for (ItemStack itemStack : item.get(p.getUniqueId().toString())) {
                    e.getInventory().setItem(num, itemStack);
                    num = num + 1;
                }
            }
        }
    }

    @EventHandler
    public void InventoryCloseEvent(InventoryCloseEvent e) {
        Player p = (Player) e.getPlayer();
        if (e.getInventory() == inv) {
            ItemStack itemStack[] = new ItemStack[36];
            item.put(p.getUniqueId().toString(), new ItemStack[36]);
            for (int i = 0; i <= 35; i++) {
                itemStack[i] = e.getInventory().getItem(i);
            }
            item.put(p.getUniqueId().toString(), itemStack);
        }
    }
}

class openInventory extends Event {
    private static final HandlerList HANDLERS = new HandlerList();
    public Player player;

    public openInventory(Player player) {
        this.player = player;
    }

    public HandlerList getHandlers() {
        return HANDLERS;
    }

    public static HandlerList getHandlerList() {
        return HANDLERS;
    }
}
```


**Plugin.yml**

```java
main: Main
name: "Virtual-Inventory"
version: 1.0
api-version: 1.16
commands:
  inventory:
```
  
  
**Config.yml**

```java
//None
```
