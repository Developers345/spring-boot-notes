# Nested IOC Container in SpringApplication

## Overview
If we want to manage dependencies **across multiple IOC containers**, we need to use **Nested Bean Factories**.  
In Spring Boot, since `SpringApplication` creates only one IOC container, we must **customize `SpringApplicationBuilder`** to create **two IOC containers** (Parent + Child).

---

## Example Program for Spring Boot Nested IOC Container

---

## `Game.java`
```java
public class Game {

    private String name;
    private String team;
    private Player player;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getTeam() {
        return team;
    }

    public void setTeam(String team) {
        this.team = team;
    }

    public Player getPlayer() {
        return player;
    }

    public void setPlayer(Player player) {
        this.player = player;
    }

    @Override
    public String toString() {
        return "Game{" +
                "name='" + name + '\'' +
                ", team='" + team + '\'' +
                ", player=" + player +
                '}';
    }
}
````

---

## `Player.java`

```java
public class Player {

    private String uniquePlayerName;

    public String getUniquePlayerName() {
        return uniquePlayerName;
    }

    public void setUniquePlayerName(String uniquePlayerName) {
        this.uniquePlayerName = uniquePlayerName;
    }

    @Override
    public String toString() {
        return "Player{" +
                "uniquePlayerName='" + uniquePlayerName + '\'' +
                '}';
    }
}
```

---

## `PlayerConfig.java`

### (IOC Container 1 – Parent Container)

```java
@Configuration
public class PlayerConfig {

    @Autowired
    private Environment env;

    @Bean
    public Player player() {
        Player player = new Player();
        player.setUniquePlayerName(env.getProperty("uniquePlayerName"));
        return player;
    }
}
```

---

## `NestedIOCContainersApplication.java`

### (IOC Container 2 – Child Container)

```java
@SpringBootApplication
public class NestedIOCContainersApplication {

    @Autowired
    private Environment env;

    @Bean
    public Game game(Player player) {
        Game game = new Game();
        game.setName(env.getProperty("name"));
        game.setTeam(env.getProperty("team"));
        game.setPlayer(player);
        return game;
    }

    public static void main(String[] args) {

        /*
         * The below code will NOT work
         */
        /*
        ApplicationContext context  =
                new SpringApplicationBuilder(NestedIOCContainersApplication.class)
                .parent(PlayerConfig.class)
                .build()
                .run(args);
        */

        /*
         * The below code WILL work
         */
        ApplicationContext context =
                new SpringApplicationBuilder(PlayerConfig.class)
                .sources(NestedIOCContainersApplication.class)
                .build()
                .run(args);

        // (or)
        /*
        ApplicationContext context = new SpringApplicationBuilder()
                .parent(PlayerConfig.class)
                .sources(NestedIOCContainersApplication.class)
                .build()
                .run(args);
        */

        Game game = context.getBean("game", Game.class);
        System.out.println(game);
    }
}
```

---

## Note

* `SpringApplication` **creates only one Environment object**.
* In Nested IOC Containers, **the same Environment object is shared** between both the Parent and Child IOC containers.
* Spring Boot **does NOT create two separate Environment objects**.

## Pictorial Representation:

<img width="1038" height="514" alt="Screenshot 2025-12-05 112252" src="https://github.com/user-attachments/assets/5f7b9e74-0e24-45ca-be39-60dca6a63914" />

![nested_ioc_containers](https://github.com/user-attachments/assets/fd3654e0-fe7f-4947-8998-7a7710634cf3)


