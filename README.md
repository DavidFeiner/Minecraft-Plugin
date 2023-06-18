package com.david.fightplugin;
import org.bukkit.*;
import org.bukkit.enchantments.Enchantment;
import org.bukkit.entity.Creature;
import org.bukkit.entity.Entity;
import org.bukkit.entity.EntityType;
import org.bukkit.entity.Player;
import org.bukkit.event.EventHandler;
import org.bukkit.event.Listener;
import org.bukkit.event.entity.EntityDeathEvent;
import org.bukkit.event.player.AsyncPlayerChatEvent;
import org.bukkit.inventory.ItemStack;
import org.bukkit.inventory.meta.ItemMeta;
import org.bukkit.plugin.java.JavaPlugin;
import org.bukkit.scheduler.BukkitRunnable;
import org.joml.Math;


public final class FightPlugin extends JavaPlugin implements Listener {

    @Override
    public void onEnable() {
        // Plugin startup logic
        System.out.println("the Fight Simulator has gotten started Hello!");
        Bukkit.getPluginManager().registerEvents(this, this);
    }

    @EventHandler
    public void onPlayerChat(AsyncPlayerChatEvent event) {
        Player player = event.getPlayer();
        String message = event.getMessage();

        if (message.equalsIgnoreCase("Next Wave")) { //Check for User Chat input
            new BukkitRunnable() {

                int mobCount = 0;
                int deadCount = 0;
                @Override
                public void run() {


                    Location loc = player.getLocation();
                    World world = player.getWorld();
                    double distance = 25;
                    for (EntityType type : new EntityType[] {EntityType.ZOMBIE, EntityType.SKELETON, EntityType.SPIDER}) {
                        double angle = Math.random() * 2 * Math.PI; //Calculating where mobs will spawn
                        double x = loc.getX() + distance * Math.cos(angle);
                        double z = loc.getZ() + distance * Math.sin(angle);
                        loc.setX(x);
                        loc.setZ(z);
                        Entity entity = world.spawnEntity(loc, type);
                        if (entity instanceof Creature) {
                            Creature creature = (Creature) entity;
                            creature.setTarget(player);
                            mobCount++;
                        }

                    }
                    LoadoutManager.giveLoadout(player);
                }



                @Override
                public void cancel() { //Checking if Mobs have been killed
                    super.cancel();
                    player.playSound(player.getLocation(), Sound.ENTITY_PLAYER_LEVELUP, 1, 1);
                    player.sendMessage(ChatColor.GREEN + "Wave completed!");
                }


            }.runTaskTimer(this, 0, 150);

            player.playSound(player.getLocation(), Sound.ENTITY_ARROW_HIT_PLAYER, 1, 1);
            player.sendMessage(ChatColor.GREEN + "Wave spawned!"); //Sending Messages when Wave has started
            rainbowMessage(player, "Mobs Spawned!");
        }
    }



    private void rainbowMessage(Player player, String message) { //Creating a rainbow pattern for "Mobs Spawned!"
        char[] colors = {'c', '6', 'e', 'a', 'b', 'd'}; // rainbow colors in order
        StringBuilder builder = new StringBuilder();
        int colorIndex = 0;
        for (char c : message.toCharArray()) {
            if (colorIndex >= colors.length) {
                colorIndex = 0;
            }
            builder.append(ChatColor.COLOR_CHAR).append(colors[colorIndex]).append(c);
            colorIndex++;
        }
        player.sendMessage(builder.toString());

    }

    private static class LoadoutManager { //Giving the player a loadout to win against the Wave
        public static void giveLoadout(Player player) {

            player.getInventory().clear();


            ItemStack helmet = createItem(Material.DIAMOND_HELMET);
            helmet.addEnchantment(Enchantment.PROTECTION_ENVIRONMENTAL, 4);
            helmet.addEnchantment(Enchantment.OXYGEN, 4);
            player.getInventory().setHelmet(helmet);

            ItemStack chestplate = createItem(Material.DIAMOND_CHESTPLATE);
            chestplate.addEnchantment(Enchantment.PROTECTION_ENVIRONMENTAL, 4);
            player.getInventory().setChestplate(chestplate);

            ItemStack leggings = createItem(Material.DIAMOND_LEGGINGS);
            leggings.addEnchantment(Enchantment.PROTECTION_ENVIRONMENTAL, 4);
            player.getInventory().setLeggings(leggings);

            ItemStack boots = createItem(Material.DIAMOND_BOOTS);
            boots.addEnchantment(Enchantment.PROTECTION_ENVIRONMENTAL, 4);
            player.getInventory().setBoots(boots);


            ItemStack sword = createItem(Material.DIAMOND_SWORD);
            sword.addEnchantment(Enchantment.DAMAGE_ALL, 5);
            player.getInventory().addItem(sword);

            ItemStack bow = createItem(Material.BOW);
            bow.addEnchantment(Enchantment.ARROW_DAMAGE, 5);
            bow.addEnchantment(Enchantment.ARROW_KNOCKBACK, 50);
            player.getInventory().addItem(bow);


            ItemStack arrows = new ItemStack(Material.ARROW, 64);
            player.getInventory().addItem(arrows);


            ItemStack food = createItem(Material.COOKED_BEEF, 64);
            player.getInventory().addItem(food);
        }

        private static ItemStack createItem(Material material) {
            return new ItemStack(material);
        }

        private static ItemStack createItem(Material material, int amount) {
            return new ItemStack(material, amount);
        }

        private static ItemStack createItem(Material material, int amount, String displayName) {
            ItemStack item = new ItemStack(material, amount);
            ItemMeta meta = item.getItemMeta();
            meta.setDisplayName(displayName);
            item.setItemMeta(meta);
            return item;
        }
    }


    @Override
    public void onDisable() {
        // Plugin shutdown logic
        System.out.println("Plugin has been stopped Bey!!!");

    }
}
