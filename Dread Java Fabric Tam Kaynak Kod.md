# DREAD — Java Edition (Fabric 1.21.1 + GeckoLib 4) Tam Kaynak Kod Paketi

Bu doküman, **DREAD** korku modunun **Java Edition (Fabric 1.21.1)** için hiçbir placeholder, TODO veya eksik parça bırakılmadan stüdyo standardında kodlanmış **62 tam dosyalık** kaynak kod paketini içerir.

## Mimari Kapsam ve Doğrulama Özeti

| Modül | Kaynak Sınıflar / Dosyalar | Uygulanan Mekanikler |
| :--- | :--- | :--- |
| **1. Çekirdek & Dread Motoru** | `DreadConfig`, `DreadTier`, `PlayerDreadState`, `DreadManager`, `DreadWorldPersistentState` | Gece (`Işık < 4` -> 5 sn'de +1, `Işık >= 10` -> hızlanan azalış, `4..9` Stasis) ve Gündüz (`Işık < 4` -> 10 sn'de +1, `Işık >= 4` -> hızlanan azalış) denklemleri, NBT kayıt ve uyku/ölüm kuralları. |
| **2. Heatmap & HOME BASE** | `HeatmapManager` | Her 10 saniyede aktif chunk'a `+1.0` puan, Sandık (`+5`), Fırın (`+6`) ve Yatak (`+25`) etkileşim bonusları, zamanla sönümlenme (`0.92x`) ve en yüksek puanlı chunk'ın `HOME BASE` seçimi. |
| **3. The Grinning Stalker (V1–V5)** | `GrinningStalkerEntity`, `StalkerVariant`, `StalkerPose`, `WatcherGoal`, `HitAndRunGoal`, `BaseWreckerGoal`, `BrinkOfDeathGoal`, `ExecutionerGoal` | Yalnızca yüzeyde spawn olma, `3.5m` dik yürüyüşten `1.5m` dört uzuv sürünmeye dinamik hitbox geçişi (2 blokluk geçitlerden geçme), V1 meşale söndürme/sahte ses, V2 arkadan saldırı + Darkness, V3 HOME BASE ışık/kapı/cam yıkımı, V4 `1.0 HP` yarım kalp kilidi, V5 Hardcore kontrolü. |
| **4. The Pitch Grin (Mağara Avcısı)** | `PitchGrinEntity`, `CeilingLurkGoal`, `TunnelHuntGoal`, `DreadSpawnDirector` | `Y < 55`, gökyüzü görünmeyen alan ve `Dread >= 351` şartı; `351–550` (`%12`), `551–750` (`%28`), `751–1000` (`%52`) spawn oranları; mağara tavanında asılı pusu, dört uzuv sürünme ve Darkness saldırısı. |
| **5. 4. Boyut, Hikaye & Apex Dread** | `UmbralPortalBlock`, `CharredBoneBlock`, `GrinningHeartItem`, `LostJournalItem`, `ApexDreadEntity`, `ModAdvancementCriteria` | 4x5 `Charred Bone Block` + `Grinning Heart` portalı, 16 blok sis sınırı (`BackgroundRendererMixin`), 5 parçalık Kayıp Günlük hikayesi, 3 fazlı Apex Dread Alt Boss savaşı, gerçek Java Advancement ve `Tome of the White Eyes` rehber sandığı. |
| **6. Herobrine & Son Oyun** | `HerobrineApparitionEntity`, `HerobrineTotemBlock`, `HerobrineBossEntity`, `EclipseOfTheHollowItem` | Boss öncesi `0 hasar` psikolojik takipçi (izleme, anlık görünme, jumpscare, Blindness), Overworld-only Totem doğrulaması, 3 fazlı Herobrine Boss (illüzyon çemberi, dünya manipülasyonu ve geri yükleme, yardımcı yaratık çağırma, sahte ölüm) ve efsanevi `Eclipse of the Hollow` kılıcı. |
| **7. İstemci HUD, Ses & Animasyon** | `DreadHudOverlay`, `CaveEchoSoundInstance`, `GrinningStalkerRenderer`, `PitchGrinRenderer`, `dread_entities.animation.json` | Beyazdan kırmızıya dönen ve titreyen `DREAD: 465 / 1000` barı, mağarada 2 kademeli yankı (Reverb) simülasyonu, yalnızca sırıtışı parlayan Emissive render katmanı ve 12 tam GeckoLib animasyonu. |

## Dosya: `build.gradle`
```groovy
plugins {
    id 'fabric-loom' version '1.7-SNAPSHOT'
    id 'maven-publish'
}

version = project.mod_version
group = project.maven_group

base {
    archivesName = project.archives_base_name
}

repositories {
    mavenCentral()
    maven {
        name = 'GeckoLib'
        url 'https://dl.cloudsmith.io/public/geckolib3/geckolib/maven/'
        content {
            includeGroup("software.bernie.geckolib")
        }
    }
}

dependencies {
    minecraft "com.mojang:minecraft:${project.minecraft_version}"
    mappings "net.fabricmc:yarn:${project.yarn_mappings}:v2"
    modImplementation "net.fabricmc:fabric-loader:${project.loader_version}"
    modImplementation "net.fabricmc.fabric-api:fabric-api:${project.fabric_version}"
    modImplementation "software.bernie.geckolib:geckolib-fabric-${project.minecraft_version}:${project.geckolib_version}"
}

processResources {
    inputs.property "version", project.version
    filesMatching("fabric.mod.json") {
        expand "version": project.version
    }
}

tasks.withType(JavaCompile).configureEach {
    it.options.release = 21
}

java {
    withSourcesJar()
    sourceCompatibility = JavaVersion.VERSION_21
    targetCompatibility = JavaVersion.VERSION_21
}
```

## Dosya: `gradle.properties`
```properties
org.gradle.jvmargs=-Xmx2G
minecraft_version=1.21.1
yarn_mappings=1.21.1+build.3
loader_version=0.16.5
fabric_version=0.105.0+1.21.1
geckolib_version=4.5.8
mod_version=1.0.0
maven_group=com.studio.dread
archives_base_name=dread-horror
```

## Dosya: `settings.gradle`
```groovy
pluginManagement {
    repositories {
        maven {
            name = 'Fabric'
            url = 'https://maven.fabricmc.net/'
        }
        mavenCentral()
        gradlePluginPortal()
    }
}
```

## Dosya: `src/main/java/com/studio/dread/DreadClient.java`
```java
package com.studio.dread;

import com.studio.dread.client.audio.CaveEchoSoundInstance;
import com.studio.dread.client.hud.DreadHudOverlay;
import com.studio.dread.client.render.UmbralHollowDimensionEffects;
import com.studio.dread.entity.ModEntities;
import com.studio.dread.entity.client.ApexDreadRenderer;
import com.studio.dread.entity.client.GrinningStalkerRenderer;
import com.studio.dread.entity.client.HerobrineRenderer;
import com.studio.dread.entity.client.PitchGrinRenderer;
import com.studio.dread.network.DreadSyncS2CPayload;
import com.studio.dread.network.HallucinationEventS2CPayload;
import com.studio.dread.registry.ModDimensions;
import com.studio.dread.registry.ModSounds;
import net.fabricmc.api.ClientModInitializer;
import net.fabricmc.fabric.api.client.networking.v1.ClientPlayNetworking;
import net.fabricmc.fabric.api.client.rendering.v1.DimensionRenderingRegistry;
import net.fabricmc.fabric.api.client.rendering.v1.EntityRendererRegistry;
import net.fabricmc.fabric.api.client.rendering.v1.HudRenderCallback;
import net.minecraft.entity.player.PlayerEntity;
import net.minecraft.util.math.Vec3d;

public class DreadClient implements ClientModInitializer {
    @Override
    public void onInitializeClient() {
        // HUD Kaydı
        HudRenderCallback.EVENT.register(new DreadHudOverlay());

        // GeckoLib Entity Renderer Kayıtları
        EntityRendererRegistry.register(ModEntities.GRINNING_STALKER, GrinningStalkerRenderer::new);
        EntityRendererRegistry.register(ModEntities.PITCH_GRIN, PitchGrinRenderer::new);
        EntityRendererRegistry.register(ModEntities.APEX_DREAD, ApexDreadRenderer::new);
        EntityRendererRegistry.register(ModEntities.HEROBRINE_APPARITION, HerobrineRenderer::new);
        EntityRendererRegistry.register(ModEntities.HEROBRINE_BOSS, HerobrineRenderer::new);

        // 4. Boyut (The Umbral Hollow) Atmosfer ve Sis Kaydı
        DimensionRenderingRegistry.registerDimensionEffects(
                ModDimensions.UMBRAL_HOLLOW_ID,
                new UmbralHollowDimensionEffects()
        );

        // S2C Ağ Paket Dinleyicileri
        ClientPlayNetworking.registerGlobalReceiver(DreadSyncS2CPayload.ID, (payload, context) -> {
            context.client().execute(() -> DreadHudOverlay.updateClientDread(payload.dreadScore()));
        });

        ClientPlayNetworking.registerGlobalReceiver(HallucinationEventS2CPayload.ID, (payload, context) -> {
            context.client().execute(() -> {
                PlayerEntity player = context.player();
                if (player == null) return;
                Vec3d behind = player.getPos().subtract(player.getRotationVec(1.0f).normalize().multiply(6.0));

                switch (payload.getEventType()) {
                    case FAKE_FOOTSTEP -> CaveEchoSoundInstance.playWithCaveEcho(
                            ModSounds.AMBIENT_FAKE_FOOTSTEP, behind.x, behind.y, behind.z, 0.85f, 0.95f
                    );
                    case FAKE_TORCH_FLICKER -> CaveEchoSoundInstance.playWithCaveEcho(
                            ModSounds.AMBIENT_FAKE_TORCH_HISS, behind.x, behind.y, behind.z, 0.80f, 0.85f
                    );
                    case STALKER_JUMPSCARE -> {
                        DreadHudOverlay.triggerJumpscareOverlay(25);
                        player.playSound(ModSounds.STALKER_JUMPSCARE, 1.4f, 1.0f);
                    }
                    case HEROBRINE_FLASH -> {
                        DreadHudOverlay.triggerJumpscareOverlay(12);
                        player.playSound(ModSounds.HEROBRINE_JUMPSCARE_STATIC, 0.9f, 1.1f);
                    }
                    case HEROBRINE_JUMPSCARE -> {
                        DreadHudOverlay.triggerJumpscareOverlay(35);
                        player.playSound(ModSounds.HEROBRINE_JUMPSCARE_STATIC, 1.6f, 0.9f);
                    }
                    case ABYSSAL_GAZE_WARNING -> DreadHudOverlay.triggerAbyssalGazeWarning(60);
                }
            });
        });
    }
}
```

## Dosya: `src/main/java/com/studio/dread/DreadMod.java`
```java
package com.studio.dread;

import com.studio.dread.config.DreadConfig;
import com.studio.dread.core.director.DreadSpawnDirector;
import com.studio.dread.core.dread.DreadManager;
import com.studio.dread.core.heatmap.HeatmapManager;
import com.studio.dread.core.persistence.DreadWorldPersistentState;
import com.studio.dread.entity.ModEntities;
import com.studio.dread.network.ModPackets;
import com.studio.dread.registry.ModAdvancementCriteria;
import com.studio.dread.registry.ModBlocks;
import com.studio.dread.registry.ModDimensions;
import com.studio.dread.registry.ModItems;
import com.studio.dread.registry.ModSounds;
import net.fabricmc.api.ModInitializer;
import net.fabricmc.fabric.api.event.lifecycle.v1.ServerTickEvents;
import net.fabricmc.fabric.api.loot.v3.LootTableEvents;
import net.fabricmc.fabric.api.networking.v1.ServerPlayConnectionEvents;
import net.minecraft.loot.LootPool;
import net.minecraft.loot.LootTables;
import net.minecraft.loot.condition.RandomChanceLootCondition;
import net.minecraft.loot.entry.ItemEntry;
import net.minecraft.loot.provider.number.ConstantLootNumberProvider;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class DreadMod implements ModInitializer {
    public static final Logger LOGGER = LoggerFactory.getLogger(DreadConfig.MOD_ID);

    @Override
    public void onInitialize() {
        LOGGER.info("Initializing DREAD Horror Mod (Java Edition Fabric 1.21.1)...");

        ModSounds.init();
        ModBlocks.init();
        ModItems.init();
        ModDimensions.init();
        ModAdvancementCriteria.init();
        ModEntities.init();
        ModPackets.registerCommonPayloads();
        HeatmapManager.registerCallbacks();

        // Oyuncu sunucuya bağlandığında anlık Dread durumunu senkronize et
        ServerPlayConnectionEvents.JOIN.register((handler, sender, server) -> {
            DreadWorldPersistentState state = DreadWorldPersistentState.get(server);
            DreadManager.forceSync(handler.getPlayer(), state.getPlayerState(handler.getPlayer().getUuid()));
        });

        // Kademeli Sunucu Oyun Döngüsü (Tick Pipeline)
        ServerTickEvents.END_SERVER_TICK.register(server -> {
            long ticks = server.getTicks();

            // Her 20 tick'te (1.0 sn): Dread ışık & zaman hesaplaması
            if (ticks % 20 == 0) {
                DreadManager.tickSecond(server);
            }

            // Her 100 tick'te (5.0 sn): AI Director (Yüzey Stalker, Yeraltı Pitch Grin ve Herobrine Haunting)
            if (ticks % 100 == 0) {
                DreadSpawnDirector.tickDirector(server, ticks);
            }

            // Her 200 tick'te (10.0 sn): Heatmap güncellemesi ve HOME BASE seçimi
            if (ticks % 200 == 0) {
                HeatmapManager.tickTenSeconds(server, ticks);
            }
        });

        // Yapı sandıklarına (Abandoned Observatory, Sealed Shelter ve antik kalıntılar) 5 parçalık Kayıp Günlük ve Grinning Heart ekleme
        LootTableEvents.MODIFY.register((key, tableBuilder, source, registries) -> {
            if (LootTables.SIMPLE_DUNGEON_CHEST.equals(key)
                    || LootTables.ABANDONED_MINESHAFT_CHEST.equals(key)
                    || LootTables.STRONGHOLD_LIBRARY_CHEST.equals(key)
                    || LootTables.ANCIENT_CITY_CHEST.equals(key)) {
                LootPool.Builder journalPool = LootPool.builder()
                        .rolls(ConstantLootNumberProvider.create(1.0f))
                        .conditionally(RandomChanceLootCondition.builder(0.65f))
                        .with(ItemEntry.builder(ModItems.LOST_JOURNAL_1).weight(25))
                        .with(ItemEntry.builder(ModItems.LOST_JOURNAL_2).weight(22))
                        .with(ItemEntry.builder(ModItems.LOST_JOURNAL_3).weight(20))
                        .with(ItemEntry.builder(ModItems.LOST_JOURNAL_4).weight(18))
                        .with(ItemEntry.builder(ModItems.LOST_JOURNAL_5).weight(15));
                tableBuilder.pool(journalPool);

                LootPool.Builder heartPool = LootPool.builder()
                        .rolls(ConstantLootNumberProvider.create(1.0f))
                        .conditionally(RandomChanceLootCondition.builder(0.30f))
                        .with(ItemEntry.builder(ModItems.GRINNING_HEART).weight(10))
                        .with(ItemEntry.builder(ModItems.CHARRED_BONE_SHARD).weight(30));
                tableBuilder.pool(heartPool);
            }
        });

        LOGGER.info("DREAD Horror Mod initialized successfully.");
    }
}
```

## Dosya: `src/main/java/com/studio/dread/block/CharredBoneBlock.java`
```java
package com.studio.dread.block;

import net.minecraft.block.PillarBlock;

public class CharredBoneBlock extends PillarBlock {
    public CharredBoneBlock(Settings settings) {
        super(settings);
    }
}
```

## Dosya: `src/main/java/com/studio/dread/block/HerobrineTotemBlock.java`
```java
package com.studio.dread.block;

import com.studio.dread.entity.ModEntities;
import com.studio.dread.entity.boss.HerobrineBossEntity;
import com.studio.dread.registry.ModAdvancementCriteria;
import com.studio.dread.registry.ModSounds;
import net.minecraft.block.Block;
import net.minecraft.block.BlockState;
import net.minecraft.block.Blocks;
import net.minecraft.entity.EntityType;
import net.minecraft.entity.LightningEntity;
import net.minecraft.entity.player.PlayerEntity;
import net.minecraft.item.ItemStack;
import net.minecraft.item.Items;
import net.minecraft.server.network.ServerPlayerEntity;
import net.minecraft.server.world.ServerWorld;
import net.minecraft.sound.SoundCategory;
import net.minecraft.text.Text;
import net.minecraft.util.Formatting;
import net.minecraft.util.Hand;
import net.minecraft.util.ItemActionResult;
import net.minecraft.util.hit.BlockHitResult;
import net.minecraft.util.math.BlockPos;
import net.minecraft.util.math.Direction;
import net.minecraft.world.World;

public class HerobrineTotemBlock extends Block {
    public HerobrineTotemBlock(Settings settings) {
        super(settings);
    }

    @Override
    protected ItemActionResult onUseWithItem(ItemStack stack, BlockState state, World world, BlockPos pos,
                                             PlayerEntity player, Hand hand, BlockHitResult hit) {
        if (!stack.isOf(Items.FLINT_AND_STEEL) && !stack.isOf(Items.FIRE_CHARGE)) {
            return ItemActionResult.PASS_TO_DEFAULT_BLOCK_ACTION;
        }

        if (world.isClient()) {
            return ItemActionResult.SUCCESS;
        }

        ServerWorld serverWorld = (ServerWorld) world;
        ServerPlayerEntity serverPlayer = (ServerPlayerEntity) player;

        // ZORUNLU KURAL: Herobrine Totemi SADECE OVERWORLD'de çalışır!
        if (!serverWorld.getRegistryKey().equals(World.OVERWORLD)) {
            serverPlayer.sendMessage(
                    Text.translatable("message.dread.totem_overworld_only").formatted(Formatting.DARK_RED, Formatting.BOLD),
                    true
            );
            serverWorld.playSound(null, pos, ModSounds.AMBIENT_FAKE_TORCH_HISS, SoundCategory.BLOCKS, 1.0f, 0.5f);
            return ItemActionResult.FAIL;
        }

        // Totem yapı doğrulaması:
        // Alt katman (Y-1): 3x3 Gold Block
        // Orta katman (Y): Merkezde bu blok, 4 yanında Redstone Torch
        // Üst katman (Y+1): Netherrack
        if (!validateTotemStructure(serverWorld, pos)) {
            serverPlayer.sendMessage(
                    Text.translatable("message.dread.totem_invalid_structure").formatted(Formatting.RED),
                    true
            );
            return ItemActionResult.FAIL;
        }

        // Üstteki Netherrack'i ateşle, meşaleleri ruh meşalesine çevir ve yıldırım düşür
        BlockPos netherrackPos = pos.up();
        serverWorld.setBlockState(netherrackPos.up(), Blocks.SOUL_FIRE.getDefaultState(), Block.NOTIFY_ALL);

        for (Direction dir : Direction.Type.HORIZONTAL) {
            BlockPos torchPos = pos.offset(dir);
            if (serverWorld.getBlockState(torchPos).isOf(Blocks.REDSTONE_TORCH)) {
                serverWorld.setBlockState(torchPos, Blocks.SOUL_TORCH.getDefaultState(), Block.NOTIFY_ALL);
            }
        }

        LightningEntity lightning = EntityType.LIGHTNING_BOLT.create(serverWorld);
        if (lightning != null) {
            lightning.refreshPositionAfterTeleport(pos.getX() + 0.5, pos.getY() + 1.0, pos.getZ() + 0.5);
            lightning.setCosmetic(true);
            serverWorld.spawnEntity(lightning);
        }

        // Hava durumunu fırtına ve geceye çevir
        serverWorld.setWeather(0, 12000, true, true);
        serverWorld.setTimeOfDay(18000L);
        serverWorld.playSound(null, pos, ModSounds.HEROBRINE_TOTEM_AWAKEN, SoundCategory.HOSTILE, 2.0f, 0.8f);

        // Herobrine Boss Fight başlat
        HerobrineBossEntity boss = ModEntities.HEROBRINE_BOSS.create(serverWorld);
        if (boss != null) {
            boss.refreshPositionAndAngles(pos.getX() + 0.5, pos.getY() + 2.5, pos.getZ() + 0.5, serverPlayer.getYaw() + 180.0f, 0.0f);
            boss.initializeArena(pos);
            boss.setTarget(serverPlayer);
            serverWorld.spawnEntity(boss);
        }

        ModAdvancementCriteria.AWAKEN_HEROBRINE_TOTEM.trigger(serverPlayer);
        return ItemActionResult.SUCCESS;
    }

    public static boolean validateTotemStructure(ServerWorld world, BlockPos centerPos) {
        // Üstte Netherrack kontrolü
        if (!world.getBlockState(centerPos.up()).isOf(Blocks.NETHERRACK)) {
            return false;
        }

        // 4 yatay yönde Redstone Torch kontrolü
        for (Direction dir : Direction.Type.HORIZONTAL) {
            BlockState side = world.getBlockState(centerPos.offset(dir));
            if (!side.isOf(Blocks.REDSTONE_TORCH) && !side.isOf(Blocks.REDSTONE_WALL_TORCH)) {
                return false;
            }
        }

        // Altta 3x3 Gold Block taban kontrolü
        BlockPos baseCenter = centerPos.down();
        for (int dx = -1; dx <= 1; dx++) {
            for (int dz = -1; dz <= 1; dz++) {
                if (!world.getBlockState(baseCenter.add(dx, 0, dz)).isOf(Blocks.GOLD_BLOCK)) {
                    return false;
                }
            }
        }
        return true;
    }
}
```

## Dosya: `src/main/java/com/studio/dread/block/UmbralPortalBlock.java`
```java
package com.studio.dread.block;

import com.studio.dread.registry.ModAdvancementCriteria;
import com.studio.dread.registry.ModBlocks;
import com.studio.dread.registry.ModDimensions;
import com.studio.dread.registry.ModSounds;
import net.minecraft.block.Block;
import net.minecraft.block.BlockState;
import net.minecraft.block.Blocks;
import net.minecraft.block.ShapeContext;
import net.minecraft.entity.Entity;
import net.minecraft.particle.ParticleTypes;
import net.minecraft.server.MinecraftServer;
import net.minecraft.server.network.ServerPlayerEntity;
import net.minecraft.server.world.ServerWorld;
import net.minecraft.sound.SoundCategory;
import net.minecraft.state.StateManager;
import net.minecraft.state.property.EnumProperty;
import net.minecraft.state.property.Properties;
import net.minecraft.util.math.BlockPos;
import net.minecraft.util.math.Direction;
import net.minecraft.util.math.Vec3d;
import net.minecraft.util.math.random.Random;
import net.minecraft.util.shape.VoxelShape;
import net.minecraft.world.BlockView;
import net.minecraft.world.TeleportTarget;
import net.minecraft.world.World;

import java.util.Set;

public class UmbralPortalBlock extends Block {
    public static final EnumProperty<Direction.Axis> AXIS = Properties.HORIZONTAL_AXIS;
    private static final VoxelShape X_SHAPE = Block.createCuboidShape(0.0, 0.0, 6.0, 16.0, 16.0, 10.0);
    private static final VoxelShape Z_SHAPE = Block.createCuboidShape(6.0, 0.0, 0.0, 10.0, 16.0, 16.0);

    public UmbralPortalBlock(Settings settings) {
        super(settings);
        setDefaultState(getStateManager().getDefaultState().with(AXIS, Direction.Axis.X));
    }

    @Override
    protected VoxelShape getOutlineShape(BlockState state, BlockView world, BlockPos pos, ShapeContext context) {
        return state.get(AXIS) == Direction.Axis.Z ? Z_SHAPE : X_SHAPE;
    }

    @Override
    protected void onEntityCollision(BlockState state, World world, BlockPos pos, Entity entity) {
        if (world.isClient() || !(entity instanceof ServerPlayerEntity player)) {
            return;
        }
        if (player.hasVehicle() || player.hasPassengers() || !player.canUsePortals(true)) {
            return;
        }

        MinecraftServer server = player.getServer();
        if (server == null) return;

        boolean inUmbral = world.getRegistryKey().equals(ModDimensions.UMBRAL_HOLLOW_WORLD_KEY);
        ServerWorld targetWorld = server.getWorld(inUmbral ? World.OVERWORLD : ModDimensions.UMBRAL_HOLLOW_WORLD_KEY);
        if (targetWorld == null) return;

        BlockPos targetPos = findOrBuildSafeLanding(targetWorld, player.getBlockPos());
        player.teleport(
                targetWorld,
                targetPos.getX() + 0.5,
                targetPos.getY() + 1.0,
                targetPos.getZ() + 0.5,
                Set.of(),
                player.getYaw(),
                player.getPitch()
        );

        targetWorld.playSound(null, targetPos, ModSounds.STALKER_VANISH, SoundCategory.PORTALS, 1.0f, 0.7f);
        if (!inUmbral) {
            ModAdvancementCriteria.ENTER_UMBRAL_HOLLOW.trigger(player);
        }
    }

    /**
     * 4x5 Charred Bone Block portal çerçevesini doğrular ve aktif eder.
     */
    public static boolean tryLightPortal(World world, BlockPos insideBottomPos) {
        if (tryActivateAlongAxis(world, insideBottomPos, Direction.Axis.X)) {
            return true;
        }
        return tryActivateAlongAxis(world, insideBottomPos, Direction.Axis.Z);
    }

    private static boolean tryActivateAlongAxis(World world, BlockPos origin, Direction.Axis axis) {
        Direction right = axis == Direction.Axis.X ? Direction.EAST : Direction.SOUTH;
        // Sol alt iç köşeyi bul (en fazla 2 blok sola kay)
        BlockPos bottomLeft = origin;
        for (int i = 0; i < 2; i++) {
            if (world.getBlockState(bottomLeft.offset(right.getOpposite())).isAir()) {
                bottomLeft = bottomLeft.offset(right.getOpposite());
            } else {
                break;
            }
        }

        // 2 genişlik x 3 yükseklik iç boşluk ve etrafındaki 4x5 Charred Bone çerçeve kontrolü
        for (int x = -1; x <= 2; x++) {
            for (int y = -1; y <= 3; y++) {
                BlockPos checkPos = bottomLeft.offset(right, x).up(y);
                boolean isBorder = (x == -1 || x == 2 || y == -1 || y == 3);
                BlockState checkState = world.getBlockState(checkPos);

                if (isBorder) {
                    if (!checkState.isOf(ModBlocks.CHARRED_BONE_BLOCK)) {
                        return false;
                    }
                } else {
                    if (!checkState.isAir()) {
                        return false;
                    }
                }
            }
        }

        // Çerçeve geçerli: 2x3 portal bloklarını yerleştir
        BlockState portalState = ModBlocks.UMBRAL_PORTAL_BLOCK.getDefaultState().with(AXIS, axis);
        for (int x = 0; x < 2; x++) {
            for (int y = 0; y < 3; y++) {
                world.setBlockState(bottomLeft.offset(right, x).up(y), portalState, Block.NOTIFY_ALL);
            }
        }
        world.playSound(null, bottomLeft, ModSounds.APEX_DREAD_ROAR, SoundCategory.BLOCKS, 0.9f, 0.55f);
        return true;
    }

    private static BlockPos findOrBuildSafeLanding(ServerWorld targetWorld, BlockPos sourcePos) {
        int clampedY = Math.max(64, Math.min(100, sourcePos.getY()));
        BlockPos center = new BlockPos(sourcePos.getX(), clampedY, sourcePos.getZ());

        // 5x5 güvenli Charred Bone platformu ve üstünde 4 blokluk temiz hava oluştur
        for (int dx = -2; dx <= 2; dx++) {
            for (int dz = -2; dz <= 2; dz++) {
                BlockPos floor = center.add(dx, 0, dz);
                if (!targetWorld.getBlockState(floor).isSolidBlock(targetWorld, floor)) {
                    targetWorld.setBlockState(floor, ModBlocks.CHARRED_BONE_BLOCK.getDefaultState());
                }
                for (int dy = 1; dy <= 4; dy++) {
                    BlockPos airPos = floor.up(dy);
                    if (!targetWorld.getBlockState(airPos).isOf(ModBlocks.UMBRAL_PORTAL_BLOCK)) {
                        targetWorld.setBlockState(airPos, Blocks.AIR.getDefaultState());
                    }
                }
            }
        }
        return center;
    }

    @Override
    public void randomDisplayTick(BlockState state, World world, BlockPos pos, Random random) {
        for (int i = 0; i < 3; i++) {
            double px = pos.getX() + random.nextDouble();
            double py = pos.getY() + random.nextDouble();
            double pz = pos.getZ() + random.nextDouble();
            world.addParticle(ParticleTypes.SMOKE, px, py, pz, 0.0, 0.03, 0.0);
            world.addParticle(ParticleTypes.ASH, px, py, pz, 0.0, -0.02, 0.0);
        }
    }

    @Override
    protected void appendProperties(StateManager.Builder<Block, BlockState> builder) {
        builder.add(AXIS);
    }
}
```

## Dosya: `src/main/java/com/studio/dread/block/WatchingBarkBlock.java`
```java
package com.studio.dread.block;

import com.studio.dread.registry.ModSounds;
import net.minecraft.block.Block;
import net.minecraft.block.BlockState;
import net.minecraft.entity.player.PlayerEntity;
import net.minecraft.particle.ParticleTypes;
import net.minecraft.sound.SoundCategory;
import net.minecraft.util.math.BlockPos;
import net.minecraft.util.math.random.Random;
import net.minecraft.world.World;

public class WatchingBarkBlock extends Block {
    public WatchingBarkBlock(Settings settings) {
        super(settings);
    }

    @Override
    public void randomDisplayTick(BlockState state, World world, BlockPos pos, Random random) {
        if (random.nextInt(12) == 0) {
            PlayerEntity nearest = world.getClosestPlayer(pos.getX() + 0.5, pos.getY() + 0.5, pos.getZ() + 0.5, 10.0, false);
            if (nearest != null) {
                world.addParticle(ParticleTypes.ASH, pos.getX() + 0.5, pos.getY() + 0.5, pos.getZ() + 0.5, 0.0, 0.02, 0.0);
                if (random.nextInt(6) == 0) {
                    world.playSound(
                            pos.getX() + 0.5, pos.getY() + 0.5, pos.getZ() + 0.5,
                            ModSounds.STALKER_BREATHING, SoundCategory.BLOCKS,
                            0.35f, 0.65f + random.nextFloat() * 0.2f, false
                    );
                }
            }
        }
    }
}
```

## Dosya: `src/main/java/com/studio/dread/client/audio/CaveEchoSoundInstance.java`
```java
package com.studio.dread.client.audio;

import com.studio.dread.config.DreadConfig;
import net.minecraft.client.MinecraftClient;
import net.minecraft.client.sound.PositionedSoundInstance;
import net.minecraft.entity.player.PlayerEntity;
import net.minecraft.sound.SoundCategory;
import net.minecraft.sound.SoundEvent;
import net.minecraft.util.math.BlockPos;
import net.minecraft.util.math.random.Random;

public final class CaveEchoSoundInstance {
    private CaveEchoSoundInstance() {}

    /**
     * Oyuncu Y < 55 altında kapalı bir mağaradayken ses olaylarına 2 kademeli mağara yankısı ekler.
     */
    public static void playWithCaveEcho(SoundEvent sound, double x, double y, double z, float volume, float pitch) {
        MinecraftClient client = MinecraftClient.getInstance();
        if (client.world == null || client.player == null) return;

        PlayerEntity player = client.player;
        BlockPos playerPos = player.getBlockPos();
        Random random = client.world.getRandom();

        // Ana sesi oynat
        client.getSoundManager().play(new PositionedSoundInstance(
                sound, SoundCategory.HOSTILE, volume, pitch, random, x, y, z
        ));

        // Mağara yankı şartı: Y < 55 ve gökyüzü görünmüyor
        if (playerPos.getY() <= DreadConfig.PITCH_GRIN_MAX_Y && !client.world.isSkyVisible(playerPos)) {
            // 1. Yankı (8 tick gecikmeli, %45 ses, 0.88x pitch)
            client.getSoundManager().play(
                    new PositionedSoundInstance(
                            sound, SoundCategory.AMBIENT, volume * 0.45f, pitch * 0.88f, random,
                            x + 3.5, y + 1.0, z - 3.5
                    ),
                    8
            );
            // 2. Yankı (16 tick gecikmeli, %20 ses, 0.78x pitch)
            client.getSoundManager().play(
                    new PositionedSoundInstance(
                            sound, SoundCategory.AMBIENT, volume * 0.20f, pitch * 0.78f, random,
                            x - 5.0, y + 1.5, z + 5.0
                    ),
                    16
            );
        }
    }
}
```

## Dosya: `src/main/java/com/studio/dread/client/hud/DreadHudOverlay.java`
```java
package com.studio.dread.client.hud;

import com.mojang.blaze3d.systems.RenderSystem;
import com.studio.dread.core.dread.DreadTier;
import net.fabricmc.fabric.api.client.rendering.v1.HudRenderCallback;
import net.minecraft.client.MinecraftClient;
import net.minecraft.client.font.TextRenderer;
import net.minecraft.client.gui.DrawContext;
import net.minecraft.client.render.RenderTickCounter;
import net.minecraft.util.math.MathHelper;

public class DreadHudOverlay implements HudRenderCallback {
    private static float clientDreadScore = 0.0f;
    private static float smoothedDreadScore = 0.0f;
    private static int jumpscareFlashTicks = 0;
    private static int gazeWarningTicks = 0;

    public static void updateClientDread(float score) {
        clientDreadScore = MathHelper.clamp(score, 0.0f, 1000.0f);
    }

    public static float getClientDreadScore() {
        return clientDreadScore;
    }

    public static void triggerJumpscareOverlay(int durationTicks) {
        jumpscareFlashTicks = durationTicks;
    }

    public static void triggerAbyssalGazeWarning(int durationTicks) {
        gazeWarningTicks = durationTicks;
    }

    @Override
    public void onHudRender(DrawContext context, RenderTickCounter tickCounter) {
        MinecraftClient client = MinecraftClient.getInstance();
        if (client.player == null || client.options.hudHidden) {
            return;
        }

        float delta = tickCounter.getTickDelta(true);
        smoothedDreadScore = MathHelper.lerp(0.15f, smoothedDreadScore, clientDreadScore);
        int displayScore = Math.round(smoothedDreadScore);
        float ratio = MathHelper.clamp(smoothedDreadScore / 1000.0f, 0.0f, 1.0f);

        int screenWidth = context.getScaledWindowWidth();
        int screenHeight = context.getScaledWindowHeight();

        // 1. Atmosferik Vinyet (Dread yükseldikçe ekran kenarları kararır)
        DreadTier tier = DreadTier.fromScore(smoothedDreadScore);
        if (tier.getVignetteIntensity() > 0.0f) {
            int alpha = (int) (tier.getVignetteIntensity() * 190.0f);
            int vignetteColor = (alpha << 24) | 0x080000;
            context.fillGradient(0, 0, screenWidth, 36, vignetteColor, 0x00000000);
            context.fillGradient(0, screenHeight - 36, screenWidth, screenHeight, 0x00000000, vignetteColor);
        }

        // 2. Jumpscare veya Abyssal Gaze Uyarısı
        if (jumpscareFlashTicks > 0) {
            jumpscareFlashTicks--;
            int flashAlpha = Math.min(180, jumpscareFlashTicks * 12);
            context.fill(0, 0, screenWidth, screenHeight, (flashAlpha << 24) | 0x1A0000);
        }
        if (gazeWarningTicks > 0) {
            gazeWarningTicks--;
            TextRenderer tr = client.textRenderer;
            String warning = "ARKANI DÖN! SIRITIŞA BAKMA!";
            int tw = tr.getWidth(warning);
            context.drawTextWithShadow(tr, warning, (screenWidth - tw) / 2, screenHeight / 2 - 40, 0xFFCC0000);
        }

        // 3. Dread Bar Titreşim (Jitter) Hesaplaması (Dread >= 200 olduğunda başlar)
        int jitterX = 0;
        int jitterY = 0;
        if (smoothedDreadScore >= 200.0f) {
            float normalizedJitter = (smoothedDreadScore - 200.0f) / 800.0f;
            float amplitude = normalizedJitter * normalizedJitter * 3.5f;
            float time = (client.player.age + delta);
            jitterX = Math.round((float) Math.sin(time * 1.7f) * amplitude);
            jitterY = Math.round((float) Math.cos(time * 2.3f) * amplitude);
        }

        // 4. Dread Bar Çizimi (Sol Üst Köşe, Temiz ve Okunaklı Korku UI)
        int barWidth = 124;
        int barHeight = 8;
        int baseX = 12 + jitterX;
        int baseY = 12 + jitterY;

        // Dış Çerçeve ve Arka Plan
        context.fill(baseX - 1, baseY - 1, baseX + barWidth + 1, baseY + barHeight + 1, 0xDD101014);
        context.fill(baseX, baseY, baseX + barWidth, baseY + barHeight, 0xEE1E1E24);

        // Dinamik Renk: Beyaz (#FFFFFF) -> Turuncu (#FF8800) -> Kan Kırmızısı (#B30000)
        int barColor = interpolateDreadColor(ratio);
        int filledWidth = Math.round(barWidth * ratio);
        if (filledWidth > 0) {
            context.fill(baseX, baseY, baseX + filledWidth, baseY + barHeight, barColor);
        }

        // 5. Sayısal Gösterge: "DREAD: 465 / 1000"
        String text = "DREAD: " + displayScore + " / 1000";
        TextRenderer textRenderer = client.textRenderer;
        context.drawTextWithShadow(textRenderer, text, baseX, baseY + barHeight + 4, barColor);
        RenderSystem.enableBlend();
    }

    /**
     * Beyaz (#FFFFFF) -> Kehribar Turuncu (#FF8800) -> Kan Kırmızısı (#B30000) renk geçişi.
     */
    private static int interpolateDreadColor(float t) {
        int r, g, b;
        if (t <= 0.5f) {
            float local = t * 2.0f;
            r = 255;
            g = Math.round(MathHelper.lerp(local, 255.0f, 136.0f));
            b = Math.round(MathHelper.lerp(local, 255.0f, 0.0f));
        } else {
            float local = (t - 0.5f) * 2.0f;
            r = Math.round(MathHelper.lerp(local, 255.0f, 179.0f));
            g = Math.round(MathHelper.lerp(local, 136.0f, 0.0f));
            b = 0;
        }
        return (0xFF << 24) | (r << 16) | (g << 8) | b;
    }
}
```

## Dosya: `src/main/java/com/studio/dread/client/render/UmbralHollowDimensionEffects.java`
```java
package com.studio.dread.client.render;

import net.minecraft.client.render.DimensionEffects;
import net.minecraft.util.math.Vec3d;

public class UmbralHollowDimensionEffects extends DimensionEffects {
    public UmbralHollowDimensionEffects() {
        super(Float.NaN, true, SkyType.END, false, true);
    }

    @Override
    public Vec3d adjustFogColor(Vec3d color, float sunHeight) {
        // Yoğun kül grisi/siyah atmosferik sis (#0D0E12)
        return new Vec3d(0.051, 0.055, 0.071);
    }

    @Override
    public boolean useThickFog(int camX, int camY) {
        return true;
    }
}
```

## Dosya: `src/main/java/com/studio/dread/config/DreadConfig.java`
```java
package com.studio.dread.config;

public final class DreadConfig {
    public static final String MOD_ID = "dread";

    // Dread Sınırları
    public static final float MIN_DREAD = 0.0f;
    public static final float MAX_DREAD = 1000.0f;
    public static final float DEATH_FLOOR_DREAD = 300.0f;
    public static final float SLEEP_LOCK_DREAD = 500.0f;
    public static final float SLEEP_RELIEF_AMOUNT = 150.0f;

    // Dread Artış Kuralları (Işık < 4)
    public static final int DARKNESS_LIGHT_THRESHOLD = 4;
    public static final float NIGHT_DREAD_GAIN_PER_SECOND = 0.20f; // Her 5 saniyede +1 Dread
    public static final float DAY_DREAD_GAIN_PER_SECOND = 0.10f;   // Her 10 saniyede +1 Dread
    public static final float UMBRAL_HOLLOW_GAIN_MULTIPLIER = 1.50f;

    // Dread Azalış Kuralları
    public static final int NIGHT_DECAY_MIN_LIGHT = 10; // Gece 10 ve üstünde azalmaya başlar
    public static final int DAY_DECAY_MIN_LIGHT = 4;    // Gündüz 4 ve üstünde azalmaya başlar

    // Anlık Olay Dread Değişimleri
    public static final float WATCHER_GAZE_DREAD_BONUS = 15.0f;
    public static final float TORCH_BREAK_DREAD_BONUS = 10.0f;
    public static final float HIT_AND_RUN_DREAD_BONUS = 35.0f;
    public static final float PITCH_GRIN_ATTACK_DREAD_BONUS = 40.0f;
    public static final float HEROBRINE_SCARE_DREAD_BONUS = 50.0f;

    // Heatmap Parametreleri
    public static final float CHUNK_PASSIVE_HEAT_10S = 1.0f;
    public static final float CHUNK_NEIGHBOR_HEAT_10S = 0.25f;
    public static final float CHEST_USE_HEAT_BONUS = 5.0f;
    public static final float FURNACE_USE_HEAT_BONUS = 6.0f;
    public static final float BED_USE_HEAT_BONUS = 25.0f;
    public static final float HOME_BASE_MIN_HEAT_THRESHOLD = 35.0f;
    public static final float HEATMAP_DECAY_FACTOR = 0.92f;
    public static final int HEATMAP_DECAY_INTERVAL_TICKS = 12000; // 10 dakika

    // Spawn Kuralları
    public static final int STALKER_EVAL_INTERVAL_TICKS = 600;     // 30 saniye
    public static final int STALKER_PACING_COOLDOWN_TICKS = 2400;  // 120 saniye
    public static final int PITCH_GRIN_EVAL_INTERVAL_TICKS = 400;  // 20 saniye
    public static final int PITCH_GRIN_MAX_Y = 54;                 // Y < 55
    public static final float PITCH_GRIN_MIN_DREAD = 351.0f;       // Dread 351+

    // Boyut ve Görsel Sınırlar
    public static final float UMBRAL_FOG_START = 4.0f;
    public static final float UMBRAL_FOG_END = 16.0f;              // Maksimum 16 blok görüş

    private DreadConfig() {}
}
```

## Dosya: `src/main/java/com/studio/dread/core/director/DreadSpawnDirector.java`
```java
package com.studio.dread.core.director;

import com.studio.dread.config.DreadConfig;
import com.studio.dread.core.dread.PlayerDreadState;
import com.studio.dread.core.persistence.DreadWorldPersistentState;
import com.studio.dread.entity.ModEntities;
import com.studio.dread.entity.boss.HerobrineApparitionEntity;
import com.studio.dread.entity.pitchgrin.PitchGrinEntity;
import com.studio.dread.entity.stalker.GrinningStalkerEntity;
import com.studio.dread.entity.stalker.StalkerVariant;
import net.minecraft.server.MinecraftServer;
import net.minecraft.server.network.ServerPlayerEntity;
import net.minecraft.server.world.ServerWorld;
import net.minecraft.util.math.BlockPos;
import net.minecraft.util.math.Box;
import net.minecraft.util.math.ChunkPos;
import net.minecraft.util.math.Vec3d;
import net.minecraft.util.math.random.Random;
import net.minecraft.world.Heightmap;
import net.minecraft.world.LightType;
import net.minecraft.world.World;

public final class DreadSpawnDirector {
    private DreadSpawnDirector() {}

    /**
     * Her 100 tick'te (5 saniye) bir sunucu tarafından çağrılır.
     * Yüzeyde The Grinning Stalker, yeraltında (Y < 55) The Pitch Grin ve Herobrine Apparition olaylarını yönetir.
     */
    public static void tickDirector(MinecraftServer server, long currentTick) {
        DreadWorldPersistentState worldState = DreadWorldPersistentState.get(server);

        for (ServerPlayerEntity player : server.getPlayerManager().getPlayerList()) {
            if (player.isCreative() || player.isSpectator() || !player.isAlive()) {
                continue;
            }

            ServerWorld world = player.getServerWorld();
            if (!world.getRegistryKey().equals(World.OVERWORLD)) {
                continue;
            }

            PlayerDreadState state = worldState.getPlayerState(player.getUuid());
            float dread = state.getDreadScore();

            // 1. YÜZEY SPAWN KONTROLÜ: The Grinning Stalker (Yalnızca yüzeyde spawn olabilir, yeraltında olamaz)
            if (world.isSkyVisible(player.getBlockPos()) && player.getBlockY() >= world.getSeaLevel() - 2) {
                if (currentTick - state.getLastStalkerSpawnTick() >= DreadConfig.STALKER_PACING_COOLDOWN_TICKS) {
                    trySpawnSurfaceStalker(world, player, state, dread, currentTick);
                }
            }

            // 2. YERALTI SPAWN KONTROLÜ: The Pitch Grin (Y < 55, gökyüzü görünmeyecek, Dread 351+)
            if (player.getBlockY() <= DreadConfig.PITCH_GRIN_MAX_Y
                    && !world.isSkyVisible(player.getBlockPos())
                    && dread >= DreadConfig.PITCH_GRIN_MIN_DREAD) {
                trySpawnUndergroundPitchGrin(world, player, state, dread, currentTick);
            }

            // 3. HEROBRINE APPARITION (Boss öncesi 0 hasarlı psikolojik takipçi)
            if (currentTick - state.getLastHerobrineScareTick() >= 3600L) {
                trySpawnHerobrineApparition(world, player, state, dread, currentTick);
            }
        }
    }

    private static void trySpawnSurfaceStalker(ServerWorld world, ServerPlayerEntity player,
                                               PlayerDreadState state, float dread, long currentTick) {
        // Singleton kuralı: Oyuncunun 72 blok çevresinde zaten aktif bir Grinning Stalker varsa yenisini çıkarma
        Box checkArea = player.getBoundingBox().expand(72.0);
        if (!world.getEntitiesByClass(GrinningStalkerEntity.class, checkArea, GrinningStalkerEntity::isAlive).isEmpty()) {
            return;
        }

        StalkerVariant chosenVariant = selectWeightedVariant(world.getRandom(), player, state, dread);
        BlockPos spawnPos = findSurfaceSpawnPosition(world, player, chosenVariant);
        if (spawnPos == null) return;

        GrinningStalkerEntity stalker = ModEntities.GRINNING_STALKER.create(world);
        if (stalker != null) {
            stalker.refreshPositionAndAngles(spawnPos.getX() + 0.5, spawnPos.getY(), spawnPos.getZ() + 0.5, world.getRandom().nextFloat() * 360.0f, 0.0f);
            stalker.setVariant(chosenVariant);
            stalker.setTarget(player);
            world.spawnEntity(stalker);
            state.setLastStalkerSpawnTick(currentTick);
        }
    }

    /**
     * Versiyon ağırlık fonksiyonları:
     * - V1 Watcher: 0-1000 arası her seviyede çıkabilir, ancak Dread yükseldikçe çıkma ihtimali azalır.
     * - V2 Hit & Run, V3 Base Wrecker, V4 Brink of Death, V5 Executioner: Dread yükseldikçe devreye girer.
     */
    private static StalkerVariant selectWeightedVariant(Random random, ServerPlayerEntity player,
                                                        PlayerDreadState state, float dread) {
        float wWatcher = Math.max(5.0f, (float) (100.0 * Math.pow(1.0 - (dread / 1050.0), 1.8)));
        float wHitRun = dread >= 200.0f ? (float) (45.0 * Math.sin(((dread - 200.0) / 800.0) * Math.PI) + 15.0) : 0.0f;

        boolean nearHomeBase = false;
        ChunkPos home = state.getHomeBaseChunk();
        if (home != null && home.getChebyshevDistance(player.getChunkPos()) <= 4) {
            nearHomeBase = true;
        }
        float wBaseWrecker = (dread >= 350.0f && nearHomeBase) ? (60.0f * ((dread - 350.0f) / 650.0f) + 20.0f) : 0.0f;
        float wBrink = dread >= 600.0f ? (float) (70.0 * Math.pow((dread - 600.0) / 400.0, 1.3) + 15.0) : 0.0f;
        float wExecutioner = dread >= 800.0f ? (float) (95.0 * Math.pow((dread - 800.0) / 200.0, 1.5) + 25.0) : 0.0f;

        float total = wWatcher + wHitRun + wBaseWrecker + wBrink + wExecutioner;
        float roll = random.nextFloat() * total;

        if ((roll -= wWatcher) <= 0.0f) return StalkerVariant.WATCHER;
        if ((roll -= wHitRun) <= 0.0f) return StalkerVariant.HIT_AND_RUN;
        if ((roll -= wBaseWrecker) <= 0.0f) return StalkerVariant.BASE_WRECKER;
        if ((roll -= wBrink) <= 0.0f) return StalkerVariant.BRINK_OF_DEATH;
        return StalkerVariant.EXECUTIONER;
    }

    private static BlockPos findSurfaceSpawnPosition(ServerWorld world, ServerPlayerEntity player, StalkerVariant variant) {
        Random random = world.getRandom();
        Vec3d look = player.getRotationVec(1.0f).normalize();

        for (int attempt = 0; attempt < 12; attempt++) {
            double distance = (variant == StalkerVariant.HIT_AND_RUN) ? (9.0 + random.nextDouble() * 5.0) : (26.0 + random.nextDouble() * 18.0);
            double angleOffset = (variant == StalkerVariant.HIT_AND_RUN)
                    ? Math.PI + (random.nextDouble() - 0.5) * 0.6 // Tam arkada
                    : (random.nextDouble() - 0.5) * Math.PI * 1.5;

            double baseAngle = Math.atan2(look.z, look.x) + angleOffset;
            int x = (int) Math.floor(player.getX() + Math.cos(baseAngle) * distance);
            int z = (int) Math.floor(player.getZ() + Math.sin(baseAngle) * distance);
            int y = world.getTopY(Heightmap.Type.MOTION_BLOCKING_NO_LEAVES, x, z);

            BlockPos candidate = new BlockPos(x, y, z);
            // ZORUNLU KURAL: Yalnızca yüzeyde spawn olabilir (gökyüzü görünür ve deniz seviyesi üstü)
            if (y >= world.getSeaLevel() - 2
                    && world.isSkyVisible(candidate)
                    && world.getBlockState(candidate.down()).isSolidBlock(world, candidate.down())
                    && world.getBlockState(candidate).isAir()) {
                return candidate;
            }
        }
        return null;
    }

    /**
     * The Pitch Grin Spawn Oranları:
     * - 351–550: Düşük-Orta (%12 şans, 90 sn cooldown, maks 1)
     * - 551–750: Yüksek (%28 şans, 50 sn cooldown, maks 1)
     * - 751–999+: Çok Yüksek (%52 şans, 25 sn cooldown, maks 2)
     */
    private static void trySpawnUndergroundPitchGrin(ServerWorld world, ServerPlayerEntity player,
                                                     PlayerDreadState state, float dread, long currentTick) {
        float spawnChance;
        long requiredCooldownTicks;
        int maxConcurrent;

        if (dread <= 550.0f) {
            spawnChance = 0.12f;
            requiredCooldownTicks = 1800L; // 90 sn
            maxConcurrent = 1;
        } else if (dread <= 750.0f) {
            spawnChance = 0.28f;
            requiredCooldownTicks = 1000L; // 50 sn
            maxConcurrent = 1;
        } else {
            spawnChance = 0.52f;
            requiredCooldownTicks = 500L;  // 25 sn
            maxConcurrent = 2;
        }

        if (currentTick - state.getLastPitchGrinSpawnTick() < requiredCooldownTicks) {
            return;
        }

        Box checkArea = player.getBoundingBox().expand(48.0);
        int existing = world.getEntitiesByClass(PitchGrinEntity.class, checkArea, PitchGrinEntity::isAlive).size();
        if (existing >= maxConcurrent) {
            return;
        }

        if (world.getRandom().nextFloat() > spawnChance) {
            return;
        }

        BlockPos cavePos = findCaveSpawnPosition(world, player);
        if (cavePos != null) {
            PitchGrinEntity pitchGrin = ModEntities.PITCH_GRIN.create(world);
            if (pitchGrin != null) {
                pitchGrin.refreshPositionAndAngles(cavePos.getX() + 0.5, cavePos.getY(), cavePos.getZ() + 0.5, 0.0f, 0.0f);
                pitchGrin.setTarget(player);
                world.spawnEntity(pitchGrin);
                state.setLastPitchGrinSpawnTick(currentTick);
            }
        }
    }

    private static BlockPos findCaveSpawnPosition(ServerWorld world, ServerPlayerEntity player) {
        Random random = world.getRandom();
        BlockPos origin = player.getBlockPos();

        for (int attempt = 0; attempt < 20; attempt++) {
            BlockPos pos = origin.add(
                    random.nextBetween(-18, 18),
                    random.nextBetween(-6, 6),
                    random.nextBetween(-18, 18)
            );
            if (pos.getY() <= DreadConfig.PITCH_GRIN_MAX_Y
                    && pos.getSquaredDistance(origin) >= 64.0
                    && !world.isSkyVisible(pos)
                    && world.getLightLevel(LightType.BLOCK, pos) <= 3
                    && world.getBlockState(pos.down()).isSolidBlock(world, pos.down())
                    && world.getBlockState(pos).isAir()
                    && world.getBlockState(pos.up()).isAir()) {
                return pos;
            }
        }
        return null;
    }

    private static void trySpawnHerobrineApparition(ServerWorld world, ServerPlayerEntity player,
                                                    PlayerDreadState state, float dread, long currentTick) {
        if (world.getRandom().nextFloat() > (0.08f + (dread / 2500.0f))) {
            return;
        }

        BlockPos origin = player.getBlockPos();
        Vec3d look = player.getRotationVec(1.0f).normalize();
        boolean closeJumpscare = dread >= 650.0f && world.getRandom().nextFloat() < 0.25f;

        double dist = closeJumpscare ? 3.5 : 34.0;
        BlockPos spawnPos = BlockPos.ofFloored(
                player.getX() + look.x * dist,
                player.getY(),
                player.getZ() + look.z * dist
        );

        if (!world.getBlockState(spawnPos).isAir()) {
            spawnPos = world.getTopPosition(Heightmap.Type.MOTION_BLOCKING_NO_LEAVES, spawnPos);
        }

        HerobrineApparitionEntity apparition = ModEntities.HEROBRINE_APPARITION.create(world);
        if (apparition != null) {
            apparition.refreshPositionAndAngles(spawnPos.getX() + 0.5, spawnPos.getY(), spawnPos.getZ() + 0.5, 0.0f, 0.0f);
            apparition.configureHaunt(player, closeJumpscare);
            world.spawnEntity(apparition);
            state.setLastHerobrineScareTick(currentTick);
        }
    }
}
```

## Dosya: `src/main/java/com/studio/dread/core/dread/DreadManager.java`
```java
package com.studio.dread.core.dread;

import com.studio.dread.config.DreadConfig;
import com.studio.dread.core.persistence.DreadWorldPersistentState;
import com.studio.dread.network.DreadSyncS2CPayload;
import com.studio.dread.network.HallucinationEventS2CPayload;
import com.studio.dread.registry.ModAdvancementCriteria;
import com.studio.dread.registry.ModDimensions;
import com.studio.dread.registry.ModItems;
import net.fabricmc.fabric.api.networking.v1.ServerPlayNetworking;
import net.minecraft.item.ItemStack;
import net.minecraft.server.MinecraftServer;
import net.minecraft.server.network.ServerPlayerEntity;
import net.minecraft.server.world.ServerWorld;
import net.minecraft.util.math.BlockPos;
import net.minecraft.world.LightType;

public final class DreadManager {
    private DreadManager() {}

    /**
     * Her 20 tick'te (1 saniye) bir sunucu tarafından çağrılır.
     * Gece ve gündüz ışık kurallarına göre Dread puanını hesaplar ve istemciyle senkronize eder.
     */
    public static void tickSecond(MinecraftServer server) {
        DreadWorldPersistentState worldState = DreadWorldPersistentState.get(server);

        for (ServerPlayerEntity player : server.getPlayerManager().getPlayerList()) {
            if (player.isSpectator() || player.isCreative()) {
                syncIfNeeded(player, worldState.getPlayerState(player.getUuid()), worldState);
                continue;
            }

            PlayerDreadState state = worldState.getPlayerState(player.getUuid());
            ServerWorld world = player.getServerWorld();
            BlockPos eyePos = BlockPos.ofFloored(player.getEyePos());

            int blockLight = world.getLightLevel(LightType.BLOCK, eyePos);
            int skyLight = world.getLightLevel(LightType.SKY, eyePos);
            long timeOfDay = world.getTimeOfDay() % 24000L;

            boolean isUmbralHollow = world.getRegistryKey().equals(ModDimensions.UMBRAL_HOLLOW_WORLD_KEY);
            boolean isDay = !isUmbralHollow && (timeOfDay >= 0L && timeOfDay < 12500L);

            // Efektif ışık seviyesi: Gündüzleri gökyüzü veya blok ışığı, geceleri yalnızca blok ışığı
            int effectiveLight = isDay ? Math.max(blockLight, skyLight) : blockLight;

            float delta = calculatePerSecondDelta(isDay, effectiveLight, isUmbralHollow);

            // Eclipse of the Hollow son oyun silahı pasif koruması: Dread artışını %75 azaltır
            if (delta > 0.0f && hasEclipseOfTheHollow(player)) {
                delta *= 0.25f;
            }

            if (delta != 0.0f) {
                state.addDread(delta);
                worldState.markDirty();
            }

            // İlk 100 Dread başarımı kontrolü
            if (state.getDreadScore() >= 100.0f) {
                ModAdvancementCriteria.FIRST_DREAD_100.trigger(player);
            }

            // Paranoya ve üzeri kademelerde rastgele işitsel halüsinasyon tetikleme
            DreadTier tier = DreadTier.fromScore(state.getDreadScore());
            if (tier.ordinal() >= DreadTier.UNEASE.ordinal() && world.getRandom().nextFloat() < 0.035f) {
                ServerPlayNetworking.send(player, new HallucinationEventS2CPayload(
                        HallucinationEventS2CPayload.EventType.FAKE_FOOTSTEP,
                        state.getDreadScore()
                ));
            }

            syncIfNeeded(player, state, worldState);
        }
    }

    /**
     * Gece ve Gündüz Dread artış/azalış denklemleri:
     * - Gece: Işık < 4 ise her 5 sn'de +1 (+0.20/sn). Işık >= 10 ise azalmaya başlar, ışık arttıkça hızlanır.
     * - Gündüz: Işık < 4 ise her 10 sn'de +1 (+0.10/sn). Işık >= 4 ise azalmaya başlar, ışık arttıkça hızlanır.
     */
    public static float calculatePerSecondDelta(boolean isDay, int effectiveLight, boolean isUmbralHollow) {
        if (!isDay) {
            // GECE KURALLARI
            if (effectiveLight < DreadConfig.DARKNESS_LIGHT_THRESHOLD) {
                float gain = DreadConfig.NIGHT_DREAD_GAIN_PER_SECOND; // +0.20 / sn (5 saniyede +1)
                return isUmbralHollow ? gain * DreadConfig.UMBRAL_HOLLOW_GAIN_MULTIPLIER : gain;
            } else if (effectiveLight >= DreadConfig.NIGHT_DECAY_MIN_LIGHT) {
                // Işık 10..15 arasında: -0.10/sn ile -0.30/sn arası hızlanan azalış
                return -(0.10f + 0.04f * (effectiveLight - DreadConfig.NIGHT_DECAY_MIN_LIGHT));
            } else {
                // Gece Işık 4..9 arası: Stasis (Gerilim Platosu)
                return 0.0f;
            }
        } else {
            // GÜNDÜZ KURALLARI
            if (effectiveLight < DreadConfig.DARKNESS_LIGHT_THRESHOLD) {
                return DreadConfig.DAY_DREAD_GAIN_PER_SECOND; // +0.10 / sn (10 saniyede +1)
            } else {
                // Gündüz Işık 4..15 arasında: -0.08/sn ile -0.50/sn arası hızlanan azalış
                return -(0.08f + 0.038f * (effectiveLight - DreadConfig.DAY_DECAY_MIN_LIGHT));
            }
        }
    }

    public static void addEventDread(ServerPlayerEntity player, float amount) {
        if (player.isCreative() || player.isSpectator()) return;
        DreadWorldPersistentState worldState = DreadWorldPersistentState.get(player.getServer());
        PlayerDreadState state = worldState.getPlayerState(player.getUuid());
        float effectiveAmount = (amount > 0.0f && hasEclipseOfTheHollow(player)) ? amount * 0.25f : amount;
        state.addDread(effectiveAmount);
        worldState.markDirty();
        forceSync(player, state);
    }

    public static void forceSync(ServerPlayerEntity player, PlayerDreadState state) {
        ServerPlayNetworking.send(player, new DreadSyncS2CPayload(
                state.getDreadScore(),
                state.getReadJournalsMask(),
                state.getHomeBaseChunk() != null ? state.getHomeBaseChunk().x : Integer.MIN_VALUE,
                state.getHomeBaseChunk() != null ? state.getHomeBaseChunk().z : Integer.MIN_VALUE
        ));
        state.markSynced();
    }

    private static void syncIfNeeded(ServerPlayerEntity player, PlayerDreadState state, DreadWorldPersistentState worldState) {
        if (state.needsNetworkSync()) {
            forceSync(player, state);
        }
    }

    private static boolean hasEclipseOfTheHollow(ServerPlayerEntity player) {
        for (int i = 0; i < player.getInventory().size(); i++) {
            ItemStack stack = player.getInventory().getStack(i);
            if (stack.isOf(ModItems.ECLIPSE_OF_THE_HOLLOW)) {
                return true;
            }
        }
        return false;
    }
}
```

## Dosya: `src/main/java/com/studio/dread/core/dread/DreadTier.java`
```java
package com.studio.dread.core.dread;

public enum DreadTier {
    CALM(0.0f, 199.9f, 0.0f, 0),
    UNEASE(200.0f, 399.9f, 0.15f, 65),
    PARANOIA(400.0f, 649.9f, 0.35f, 85),
    TERROR(650.0f, 849.9f, 0.55f, 110),
    HYSTERIA(850.0f, 1000.0f, 0.75f, 135);

    private final float minDread;
    private final float maxDread;
    private final float vignetteIntensity;
    private final int heartbeatBpm;

    DreadTier(float minDread, float maxDread, float vignetteIntensity, int heartbeatBpm) {
        this.minDread = minDread;
        this.maxDread = maxDread;
        this.vignetteIntensity = vignetteIntensity;
        this.heartbeatBpm = heartbeatBpm;
    }

    public float getMinDread() {
        return minDread;
    }

    public float getMaxDread() {
        return maxDread;
    }

    public float getVignetteIntensity() {
        return vignetteIntensity;
    }

    public int getHeartbeatBpm() {
        return heartbeatBpm;
    }

    public static DreadTier fromScore(float score) {
        if (score >= 850.0f) return HYSTERIA;
        if (score >= 650.0f) return TERROR;
        if (score >= 400.0f) return PARANOIA;
        if (score >= 200.0f) return UNEASE;
        return CALM;
    }
}
```

## Dosya: `src/main/java/com/studio/dread/core/dread/PlayerDreadState.java`
```java
package com.studio.dread.core.dread;

import com.studio.dread.config.DreadConfig;
import net.minecraft.nbt.NbtCompound;
import net.minecraft.util.math.ChunkPos;
import net.minecraft.util.math.MathHelper;

public class PlayerDreadState {
    private float dreadScore = 0.0f;
    private float lastSyncedDread = -1.0f;
    private int readJournalsMask = 0; // Bitmask 0..31 (5 parçalık hikaye)
    private long lastStalkerSpawnTick = -24000L;
    private long lastPitchGrinSpawnTick = -24000L;
    private long lastHerobrineScareTick = -24000L;
    private long lastBlockInteractionHeatTick = 0L;
    private ChunkPos homeBaseChunk = null;
    private boolean lockedAtMaxByBoss = false;

    public float getDreadScore() {
        return lockedAtMaxByBoss ? 999.0f : dreadScore;
    }

    public int getDisplayDread() {
        return Math.round(getDreadScore());
    }

    public void setDreadScore(float value) {
        this.dreadScore = MathHelper.clamp(value, DreadConfig.MIN_DREAD, DreadConfig.MAX_DREAD);
    }

    public void addDread(float delta) {
        setDreadScore(this.dreadScore + delta);
    }

    public boolean needsNetworkSync() {
        return Math.abs(getDreadScore() - lastSyncedDread) >= 0.5f;
    }

    public void markSynced() {
        this.lastSyncedDread = getDreadScore();
    }

    public int getReadJournalsMask() {
        return readJournalsMask;
    }

    public boolean hasUnlockedJournal(int index1To5) {
        int bit = 1 << (index1To5 - 1);
        return (readJournalsMask & bit) != 0;
    }

    public void unlockJournal(int index1To5) {
        int bit = 1 << (index1To5 - 1);
        this.readJournalsMask |= bit;
    }

    public int getNextUnreadJournalIndex() {
        for (int i = 1; i <= 5; i++) {
            if (!hasUnlockedJournal(i)) {
                return i;
            }
        }
        return 5;
    }

    public long getLastStalkerSpawnTick() {
        return lastStalkerSpawnTick;
    }

    public void setLastStalkerSpawnTick(long tick) {
        this.lastStalkerSpawnTick = tick;
    }

    public long getLastPitchGrinSpawnTick() {
        return lastPitchGrinSpawnTick;
    }

    public void setLastPitchGrinSpawnTick(long tick) {
        this.lastPitchGrinSpawnTick = tick;
    }

    public long getLastHerobrineScareTick() {
        return lastHerobrineScareTick;
    }

    public void setLastHerobrineScareTick(long tick) {
        this.lastHerobrineScareTick = tick;
    }

    public long getLastBlockInteractionHeatTick() {
        return lastBlockInteractionHeatTick;
    }

    public void setLastBlockInteractionHeatTick(long tick) {
        this.lastBlockInteractionHeatTick = tick;
    }

    public ChunkPos getHomeBaseChunk() {
        return homeBaseChunk;
    }

    public void setHomeBaseChunk(ChunkPos homeBaseChunk) {
        this.homeBaseChunk = homeBaseChunk;
    }

    public void setLockedAtMaxByBoss(boolean locked) {
        this.lockedAtMaxByBoss = locked;
    }

    public NbtCompound toNbt() {
        NbtCompound nbt = new NbtCompound();
        nbt.putFloat("dreadScore", dreadScore);
        nbt.putInt("readJournalsMask", readJournalsMask);
        nbt.putLong("lastStalkerSpawnTick", lastStalkerSpawnTick);
        nbt.putLong("lastPitchGrinSpawnTick", lastPitchGrinSpawnTick);
        nbt.putLong("lastHerobrineScareTick", lastHerobrineScareTick);
        if (homeBaseChunk != null) {
            nbt.putInt("homeChunkX", homeBaseChunk.x);
            nbt.putInt("homeChunkZ", homeBaseChunk.z);
        }
        return nbt;
    }

    public static PlayerDreadState fromNbt(NbtCompound nbt) {
        PlayerDreadState state = new PlayerDreadState();
        state.setDreadScore(nbt.getFloat("dreadScore"));
        state.readJournalsMask = nbt.getInt("readJournalsMask");
        state.lastStalkerSpawnTick = nbt.getLong("lastStalkerSpawnTick");
        state.lastPitchGrinSpawnTick = nbt.getLong("lastPitchGrinSpawnTick");
        state.lastHerobrineScareTick = nbt.getLong("lastHerobrineScareTick");
        if (nbt.contains("homeChunkX") && nbt.contains("homeChunkZ")) {
            state.homeBaseChunk = new ChunkPos(nbt.getInt("homeChunkX"), nbt.getInt("homeChunkZ"));
        }
        return state;
    }
}
```

## Dosya: `src/main/java/com/studio/dread/core/heatmap/HeatmapManager.java`
```java
package com.studio.dread.core.heatmap;

import com.studio.dread.config.DreadConfig;
import com.studio.dread.core.dread.PlayerDreadState;
import com.studio.dread.core.persistence.DreadWorldPersistentState;
import it.unimi.dsi.fastutil.longs.Long2FloatMap;
import it.unimi.dsi.fastutil.longs.Long2FloatOpenHashMap;
import it.unimi.dsi.fastutil.longs.LongArrayList;
import net.fabricmc.fabric.api.event.player.UseBlockCallback;
import net.minecraft.block.AbstractFurnaceBlock;
import net.minecraft.block.BarrelBlock;
import net.minecraft.block.BedBlock;
import net.minecraft.block.Block;
import net.minecraft.block.BlockState;
import net.minecraft.block.ChestBlock;
import net.minecraft.block.EnderChestBlock;
import net.minecraft.block.ShulkerBoxBlock;
import net.minecraft.server.MinecraftServer;
import net.minecraft.server.network.ServerPlayerEntity;
import net.minecraft.util.ActionResult;
import net.minecraft.util.math.BlockPos;
import net.minecraft.util.math.ChunkPos;
import net.minecraft.world.World;

public final class HeatmapManager {
    private HeatmapManager() {}

    public static void registerCallbacks() {
        UseBlockCallback.EVENT.register((player, world, hand, hitResult) -> {
            if (world.isClient() || !(player instanceof ServerPlayerEntity serverPlayer)) {
                return ActionResult.PASS;
            }
            if (!world.getRegistryKey().equals(World.OVERWORLD)) {
                return ActionResult.PASS;
            }

            BlockPos pos = hitResult.getBlockPos();
            BlockState state = world.getBlockState(pos);
            Block block = state.getBlock();

            DreadWorldPersistentState worldState = DreadWorldPersistentState.get(serverPlayer.getServer());
            PlayerDreadState playerState = worldState.getPlayerState(serverPlayer.getUuid());
            long now = world.getTime();

            // 5 saniye (100 tick) spam koruması
            if (now - playerState.getLastBlockInteractionHeatTick() < 100L) {
                return ActionResult.PASS;
            }

            ChunkPos chunkPos = new ChunkPos(pos);
            float bonus = 0.0f;

            if (block instanceof ChestBlock || block instanceof BarrelBlock
                    || block instanceof ShulkerBoxBlock || block instanceof EnderChestBlock) {
                bonus = DreadConfig.CHEST_USE_HEAT_BONUS;
            } else if (block instanceof AbstractFurnaceBlock) {
                bonus = DreadConfig.FURNACE_USE_HEAT_BONUS;
            } else if (block instanceof BedBlock) {
                bonus = DreadConfig.BED_USE_HEAT_BONUS;
            }

            if (bonus > 0.0f) {
                playerState.setLastBlockInteractionHeatTick(now);
                worldState.addChunkHeat(chunkPos, bonus);
                recalculatePlayerHomeBase(serverPlayer, worldState);
            }

            return ActionResult.PASS;
        });
    }

    /**
     * Her 10 saniyede (200 tick) bir çağrılır.
     * Oyuncunun bulunduğu chunk'a +1 puan ekler ve en yüksek puanlı bölgeyi HOME BASE olarak işaretler.
     */
    public static void tickTenSeconds(MinecraftServer server, long currentTick) {
        DreadWorldPersistentState worldState = DreadWorldPersistentState.get(server);

        for (ServerPlayerEntity player : server.getPlayerManager().getPlayerList()) {
            if (player.isSpectator() || !player.getServerWorld().getRegistryKey().equals(World.OVERWORLD)) {
                continue;
            }

            ChunkPos center = player.getChunkPos();
            worldState.addChunkHeat(center, DreadConfig.CHUNK_PASSIVE_HEAT_10S);

            // 3x3 komşu chunk bütünlüğü
            for (int dx = -1; dx <= 1; dx++) {
                for (int dz = -1; dz <= 1; dz++) {
                    if (dx == 0 && dz == 0) continue;
                    worldState.addChunkHeat(new ChunkPos(center.x + dx, center.z + dz), DreadConfig.CHUNK_NEIGHBOR_HEAT_10S);
                }
            }

            recalculatePlayerHomeBase(player, worldState);
        }

        // Her 10 dakikada bir sönümlenme (decay) ve temizlik
        if (currentTick - worldState.getLastDecayTick() >= DreadConfig.HEATMAP_DECAY_INTERVAL_TICKS) {
            applyDecayAndPrune(worldState);
            worldState.setLastDecayTick(currentTick);
        }
    }

    public static void recalculatePlayerHomeBase(ServerPlayerEntity player, DreadWorldPersistentState worldState) {
        Long2FloatOpenHashMap map = worldState.getOverworldHeatmap();
        ChunkPos playerChunk = player.getChunkPos();

        float bestScore = DreadConfig.HOME_BASE_MIN_HEAT_THRESHOLD;
        ChunkPos bestChunk = null;

        for (Long2FloatMap.Entry entry : map.long2FloatEntrySet()) {
            float heat = entry.getFloatValue();
            if (heat >= bestScore) {
                ChunkPos candidate = new ChunkPos(entry.getLongKey());
                // Oyuncuya 32 chunk (512 blok) mesafedeki en yüksek puanlı üssü seç
                if (candidate.getChebyshevDistance(playerChunk) <= 32) {
                    bestScore = heat;
                    bestChunk = candidate;
                }
            }
        }

        if (bestChunk != null) {
            PlayerDreadState state = worldState.getPlayerState(player.getUuid());
            state.setHomeBaseChunk(bestChunk);
            worldState.markDirty();
        }
    }

    private static void applyDecayAndPrune(DreadWorldPersistentState worldState) {
        Long2FloatOpenHashMap map = worldState.getOverworldHeatmap();
        LongArrayList toRemove = new LongArrayList();

        for (Long2FloatMap.Entry entry : map.long2FloatEntrySet()) {
            float decayed = entry.getFloatValue() * DreadConfig.HEATMAP_DECAY_FACTOR;
            if (decayed < 1.0f) {
                toRemove.add(entry.getLongKey());
            } else {
                entry.setValue(decayed);
            }
        }

        for (int i = 0; i < toRemove.size(); i++) {
            map.remove(toRemove.getLong(i));
        }
    }
}
```

## Dosya: `src/main/java/com/studio/dread/core/persistence/DreadWorldPersistentState.java`
```java
package com.studio.dread.core.persistence;

import com.studio.dread.config.DreadConfig;
import com.studio.dread.core.dread.PlayerDreadState;
import it.unimi.dsi.fastutil.longs.Long2FloatMap;
import it.unimi.dsi.fastutil.longs.Long2FloatOpenHashMap;
import net.minecraft.nbt.NbtCompound;
import net.minecraft.nbt.NbtElement;
import net.minecraft.nbt.NbtList;
import net.minecraft.registry.RegistryWrapper;
import net.minecraft.server.MinecraftServer;
import net.minecraft.server.world.ServerWorld;
import net.minecraft.util.math.BlockPos;
import net.minecraft.util.math.ChunkPos;
import net.minecraft.world.PersistentState;
import net.minecraft.world.PersistentStateManager;
import net.minecraft.world.World;

import java.util.HashMap;
import java.util.Map;
import java.util.UUID;

public class DreadWorldPersistentState extends PersistentState {
    private static final String DATA_KEY = DreadConfig.MOD_ID + "_world_state";

    private final Map<UUID, PlayerDreadState> playerStates = new HashMap<>();
    private final Long2FloatOpenHashMap overworldHeatmap = new Long2FloatOpenHashMap();
    private final Map<BlockPos, String> corruptedArenaBlocks = new HashMap<>();
    private long lastDecayTick = 0L;

    public static final Type<DreadWorldPersistentState> TYPE = new Type<>(
            DreadWorldPersistentState::new,
            DreadWorldPersistentState::createFromNbt,
            null
    );

    public DreadWorldPersistentState() {
        overworldHeatmap.defaultReturnValue(0.0f);
    }

    public static DreadWorldPersistentState get(MinecraftServer server) {
        ServerWorld overworld = server.getWorld(World.OVERWORLD);
        if (overworld == null) {
            overworld = server.getOverworld();
        }
        PersistentStateManager manager = overworld.getPersistentStateManager();
        return manager.getOrCreate(TYPE, DATA_KEY);
    }

    public PlayerDreadState getPlayerState(UUID playerUuid) {
        return playerStates.computeIfAbsent(playerUuid, id -> {
            markDirty();
            return new PlayerDreadState();
        });
    }

    public Long2FloatOpenHashMap getOverworldHeatmap() {
        return overworldHeatmap;
    }

    public void addChunkHeat(ChunkPos pos, float amount) {
        long key = pos.toLong();
        float updated = overworldHeatmap.get(key) + amount;
        overworldHeatmap.put(key, updated);
        markDirty();
    }

    public float getChunkHeat(ChunkPos pos) {
        return overworldHeatmap.get(pos.toLong());
    }

    public long getLastDecayTick() {
        return lastDecayTick;
    }

    public void setLastDecayTick(long tick) {
        this.lastDecayTick = tick;
        markDirty();
    }

    public Map<BlockPos, String> getCorruptedArenaBlocks() {
        return corruptedArenaBlocks;
    }

    public void recordArenaBlockBeforeCorruption(BlockPos pos, String blockId) {
        if (!corruptedArenaBlocks.containsKey(pos)) {
            corruptedArenaBlocks.put(pos.toImmutable(), blockId);
            markDirty();
        }
    }

    @Override
    public NbtCompound writeNbt(NbtCompound nbt, RegistryWrapper.WrapperLookup registryLookup) {
        nbt.putLong("lastDecayTick", lastDecayTick);

        NbtCompound playersTag = new NbtCompound();
        for (Map.Entry<UUID, PlayerDreadState> entry : playerStates.entrySet()) {
            playersTag.put(entry.getKey().toString(), entry.getValue().toNbt());
        }
        nbt.put("players", playersTag);

        NbtList heatmapList = new NbtList();
        for (Long2FloatMap.Entry entry : overworldHeatmap.long2FloatEntrySet()) {
            if (entry.getFloatValue() >= 0.5f) {
                NbtCompound chunkTag = new NbtCompound();
                chunkTag.putLong("chunkLong", entry.getLongKey());
                chunkTag.putFloat("heat", entry.getFloatValue());
                heatmapList.add(chunkTag);
            }
        }
        nbt.put("heatmap", heatmapList);

        NbtList arenaList = new NbtList();
        for (Map.Entry<BlockPos, String> entry : corruptedArenaBlocks.entrySet()) {
            NbtCompound blockTag = new NbtCompound();
            blockTag.putLong("pos", entry.getKey().asLong());
            blockTag.putString("blockId", entry.getValue());
            arenaList.add(blockTag);
        }
        nbt.put("corruptedArenaBlocks", arenaList);

        return nbt;
    }

    public static DreadWorldPersistentState createFromNbt(NbtCompound nbt, RegistryWrapper.WrapperLookup registryLookup) {
        DreadWorldPersistentState state = new DreadWorldPersistentState();
        state.lastDecayTick = nbt.getLong("lastDecayTick");

        if (nbt.contains("players", NbtElement.COMPOUND_TYPE)) {
            NbtCompound playersTag = nbt.getCompound("players");
            for (String key : playersTag.getKeys()) {
                try {
                    UUID uuid = UUID.fromString(key);
                    state.playerStates.put(uuid, PlayerDreadState.fromNbt(playersTag.getCompound(key)));
                } catch (IllegalArgumentException ignored) {
                }
            }
        }

        if (nbt.contains("heatmap", NbtElement.LIST_TYPE)) {
            NbtList heatmapList = nbt.getList("heatmap", NbtElement.COMPOUND_TYPE);
            for (int i = 0; i < heatmapList.size(); i++) {
                NbtCompound chunkTag = heatmapList.getCompound(i);
                state.overworldHeatmap.put(chunkTag.getLong("chunkLong"), chunkTag.getFloat("heat"));
            }
        }

        if (nbt.contains("corruptedArenaBlocks", NbtElement.LIST_TYPE)) {
            NbtList arenaList = nbt.getList("corruptedArenaBlocks", NbtElement.COMPOUND_TYPE);
            for (int i = 0; i < arenaList.size(); i++) {
                NbtCompound blockTag = arenaList.getCompound(i);
                state.corruptedArenaBlocks.put(BlockPos.fromLong(blockTag.getLong("pos")), blockTag.getString("blockId"));
            }
        }

        return state;
    }
}
```

## Dosya: `src/main/java/com/studio/dread/entity/ModEntities.java`
```java
package com.studio.dread.entity;

import com.studio.dread.config.DreadConfig;
import com.studio.dread.entity.boss.ApexDreadEntity;
import com.studio.dread.entity.boss.HerobrineApparitionEntity;
import com.studio.dread.entity.boss.HerobrineBossEntity;
import com.studio.dread.entity.pitchgrin.PitchGrinEntity;
import com.studio.dread.entity.stalker.GrinningStalkerEntity;
import net.fabricmc.fabric.api.object.builder.v1.entity.FabricDefaultAttributeRegistry;
import net.minecraft.entity.EntityType;
import net.minecraft.entity.SpawnGroup;
import net.minecraft.registry.Registries;
import net.minecraft.registry.Registry;
import net.minecraft.util.Identifier;

public final class ModEntities {
    public static final EntityType<GrinningStalkerEntity> GRINNING_STALKER = Registry.register(
            Registries.ENTITY_TYPE,
            Identifier.of(DreadConfig.MOD_ID, "grinning_stalker"),
            EntityType.Builder.create(GrinningStalkerEntity::new, SpawnGroup.MONSTER)
                    .dimensions(1.4f, 3.5f)
                    .maxTrackingRange(96)
                    .build()
    );

    public static final EntityType<PitchGrinEntity> PITCH_GRIN = Registry.register(
            Registries.ENTITY_TYPE,
            Identifier.of(DreadConfig.MOD_ID, "pitch_grin"),
            EntityType.Builder.create(PitchGrinEntity::new, SpawnGroup.MONSTER)
                    .dimensions(1.3f, 1.45f)
                    .maxTrackingRange(64)
                    .build()
    );

    public static final EntityType<ApexDreadEntity> APEX_DREAD = Registry.register(
            Registries.ENTITY_TYPE,
            Identifier.of(DreadConfig.MOD_ID, "apex_dread"),
            EntityType.Builder.create(ApexDreadEntity::new, SpawnGroup.MONSTER)
                    .dimensions(2.2f, 4.5f)
                    .fireImmune()
                    .maxTrackingRange(112)
                    .build()
    );

    public static final EntityType<HerobrineApparitionEntity> HEROBRINE_APPARITION = Registry.register(
            Registries.ENTITY_TYPE,
            Identifier.of(DreadConfig.MOD_ID, "herobrine_apparition"),
            EntityType.Builder.create(HerobrineApparitionEntity::new, SpawnGroup.MISC)
                    .dimensions(0.6f, 1.8f)
                    .maxTrackingRange(96)
                    .build()
    );

    public static final EntityType<HerobrineBossEntity> HEROBRINE_BOSS = Registry.register(
            Registries.ENTITY_TYPE,
            Identifier.of(DreadConfig.MOD_ID, "herobrine_boss"),
            EntityType.Builder.create(HerobrineBossEntity::new, SpawnGroup.MONSTER)
                    .dimensions(0.75f, 2.0f)
                    .fireImmune()
                    .maxTrackingRange(128)
                    .build()
    );

    private ModEntities() {}

    public static void init() {
        FabricDefaultAttributeRegistry.register(GRINNING_STALKER, GrinningStalkerEntity.createStalkerAttributes());
        FabricDefaultAttributeRegistry.register(PITCH_GRIN, PitchGrinEntity.createPitchGrinAttributes());
        FabricDefaultAttributeRegistry.register(APEX_DREAD, ApexDreadEntity.createApexAttributes());
        FabricDefaultAttributeRegistry.register(HEROBRINE_APPARITION, HerobrineApparitionEntity.createApparitionAttributes());
        FabricDefaultAttributeRegistry.register(HEROBRINE_BOSS, HerobrineBossEntity.createHerobrineBossAttributes());
    }
}
```

## Dosya: `src/main/java/com/studio/dread/entity/boss/ApexDreadEntity.java`
```java
package com.studio.dread.entity.boss;

import com.studio.dread.core.dread.DreadManager;
import com.studio.dread.entity.ModEntities;
import com.studio.dread.entity.stalker.GrinningStalkerEntity;
import com.studio.dread.entity.stalker.StalkerVariant;
import com.studio.dread.entity.stalker.ai.BaseWreckerGoal;
import com.studio.dread.network.HallucinationEventS2CPayload;
import com.studio.dread.registry.ModAdvancementCriteria;
import com.studio.dread.registry.ModBlocks;
import com.studio.dread.registry.ModItems;
import com.studio.dread.registry.ModSounds;
import net.fabricmc.fabric.api.networking.v1.ServerPlayNetworking;
import net.minecraft.block.BlockState;
import net.minecraft.block.Blocks;
import net.minecraft.block.entity.ChestBlockEntity;
import net.minecraft.entity.EntityDimensions;
import net.minecraft.entity.EntityPose;
import net.minecraft.entity.EntityType;
import net.minecraft.entity.ai.goal.ActiveTargetGoal;
import net.minecraft.entity.ai.goal.LookAtEntityGoal;
import net.minecraft.entity.ai.goal.MeleeAttackGoal;
import net.minecraft.entity.ai.goal.SwimGoal;
import net.minecraft.entity.attribute.DefaultAttributeContainer;
import net.minecraft.entity.attribute.EntityAttributes;
import net.minecraft.entity.boss.BossBar;
import net.minecraft.entity.boss.ServerBossBar;
import net.minecraft.entity.damage.DamageSource;
import net.minecraft.entity.data.DataTracker;
import net.minecraft.entity.data.TrackedData;
import net.minecraft.entity.data.TrackedDataHandlerRegistry;
import net.minecraft.entity.effect.StatusEffectInstance;
import net.minecraft.entity.effect.StatusEffects;
import net.minecraft.entity.mob.HostileEntity;
import net.minecraft.entity.player.PlayerEntity;
import net.minecraft.item.ItemStack;
import net.minecraft.nbt.NbtCompound;
import net.minecraft.particle.ParticleTypes;
import net.minecraft.server.network.ServerPlayerEntity;
import net.minecraft.server.world.ServerWorld;
import net.minecraft.sound.SoundCategory;
import net.minecraft.sound.SoundEvent;
import net.minecraft.text.Text;
import net.minecraft.util.Formatting;
import net.minecraft.util.math.BlockPos;
import net.minecraft.util.math.Vec3d;
import net.minecraft.world.World;
import software.bernie.geckolib.animatable.GeoEntity;
import software.bernie.geckolib.animatable.instance.AnimatableInstanceCache;
import software.bernie.geckolib.animation.AnimatableManager;
import software.bernie.geckolib.animation.AnimationController;
import software.bernie.geckolib.animation.AnimationState;
import software.bernie.geckolib.animation.PlayState;
import software.bernie.geckolib.animation.RawAnimation;
import software.bernie.geckolib.util.GeckoLibUtil;

public class ApexDreadEntity extends HostileEntity implements GeoEntity {
    private static final TrackedData<Integer> BOSS_PHASE = DataTracker.registerData(ApexDreadEntity.class, TrackedDataHandlerRegistry.INTEGER);
    private static final TrackedData<Integer> GAZE_CHARGE_TICKS = DataTracker.registerData(ApexDreadEntity.class, TrackedDataHandlerRegistry.INTEGER);

    private static final EntityDimensions APEX_DIMENSIONS = EntityDimensions.changing(2.2f, 4.5f).withEyeHeight(4.0f);

    private static final RawAnimation IDLE_ANIM = RawAnimation.begin().thenLoop("animation.dread.idle");
    private static final RawAnimation WALK_ANIM = RawAnimation.begin().thenLoop("animation.dread.walk");
    private static final RawAnimation SPRINT_ANIM = RawAnimation.begin().thenLoop("animation.dread.sprint");
    private static final RawAnimation ATTACK_ANIM = RawAnimation.begin().thenPlay("animation.dread.attack");
    private static final RawAnimation PHASE_TRANSITION_1 = RawAnimation.begin().thenPlay("animation.dread.boss_phase_1_to_2");
    private static final RawAnimation PHASE_TRANSITION_2 = RawAnimation.begin().thenPlay("animation.dread.boss_phase_2_to_3");

    private final ServerBossBar bossBar = new ServerBossBar(
            Text.translatable("entity.dread.apex_dread").formatted(Formatting.DARK_RED, Formatting.BOLD),
            BossBar.Color.RED,
            BossBar.Style.NOTCHED_12
    );

    private final AnimatableInstanceCache geoCache = GeckoLibUtil.createInstanceCache(this);
    private int specialAttackCooldown = 100;
    private int transitionAnimTicks = 0;

    public ApexDreadEntity(EntityType<? extends HostileEntity> entityType, World world) {
        super(entityType, world);
        this.experiencePoints = 250;
    }

    public static DefaultAttributeContainer.Builder createApexAttributes() {
        return HostileEntity.createHostileAttributes()
                .add(EntityAttributes.GENERIC_MAX_HEALTH, 600.0)
                .add(EntityAttributes.GENERIC_MOVEMENT_SPEED, 0.34)
                .add(EntityAttributes.GENERIC_ATTACK_DAMAGE, 14.0)
                .add(EntityAttributes.GENERIC_ARMOR, 10.0)
                .add(EntityAttributes.GENERIC_FOLLOW_RANGE, 64.0)
                .add(EntityAttributes.GENERIC_KNOCKBACK_RESISTANCE, 0.95)
                .add(EntityAttributes.GENERIC_STEP_HEIGHT, 1.5);
    }

    @Override
    protected void initDataTracker(DataTracker.Builder builder) {
        super.initDataTracker(builder);
        builder.add(BOSS_PHASE, 1);
        builder.add(GAZE_CHARGE_TICKS, 0);
    }

    @Override
    protected void initGoals() {
        this.goalSelector.add(0, new SwimGoal(this));
        this.goalSelector.add(2, new MeleeAttackGoal(this, 1.35, false));
        this.goalSelector.add(7, new LookAtEntityGoal(this, PlayerEntity.class, 48.0f));
        this.targetSelector.add(1, new ActiveTargetGoal<>(this, PlayerEntity.class, true));
    }

    @Override
    protected EntityDimensions getBaseDimensions(EntityPose pose) {
        return APEX_DIMENSIONS;
    }

    public int getBossPhase() {
        return this.dataTracker.get(BOSS_PHASE);
    }

    public void setBossPhase(int phase) {
        this.dataTracker.set(BOSS_PHASE, phase);
    }

    @Override
    public void tick() {
        super.tick();
        if (this.getWorld().isClient()) {
            return;
        }

        this.bossBar.setPercent(this.getHealth() / this.getMaxHealth());
        updatePhaseProgression();

        if (transitionAnimTicks > 0) {
            transitionAnimTicks--;
            this.getNavigation().stop();
            return;
        }

        // Abyssal Gaze (Faz 3 Bakış Kilitlenmesi Saldırısı)
        int gazeTicks = this.dataTracker.get(GAZE_CHARGE_TICKS);
        if (gazeTicks > 0) {
            this.dataTracker.set(GAZE_CHARGE_TICKS, gazeTicks - 1);
            if (gazeTicks == 1) {
                executeAbyssalGazeBurst();
            }
            return;
        }

        if (--specialAttackCooldown <= 0 && this.getTarget() instanceof ServerPlayerEntity target) {
            int phase = getBossPhase();
            if (phase == 1) {
                performCrawlerChargeOrTorchPurge(target);
                specialAttackCooldown = 110;
            } else if (phase == 2) {
                performPillarAmbushOrClones(target);
                specialAttackCooldown = 95;
            } else {
                startAbyssalGazeCharge(target);
                specialAttackCooldown = 120;
            }
        }
    }

    private void updatePhaseProgression() {
        float healthRatio = this.getHealth() / this.getMaxHealth();
        int currentPhase = getBossPhase();

        if (currentPhase == 1 && healthRatio <= 0.66f) {
            setBossPhase(2);
            transitionAnimTicks = 50;
            this.playSound(ModSounds.APEX_DREAD_ROAR, 2.0f, 0.75f);
            this.bossBar.setColor(BossBar.Color.PURPLE);
        } else if (currentPhase == 2 && healthRatio <= 0.30f) {
            setBossPhase(3);
            transitionAnimTicks = 50;
            this.playSound(ModSounds.APEX_DREAD_ROAR, 2.5f, 0.55f);
            if (this.getAttributeInstance(EntityAttributes.GENERIC_MOVEMENT_SPEED) != null) {
                this.getAttributeInstance(EntityAttributes.GENERIC_MOVEMENT_SPEED).setBaseValue(0.42);
            }
            if (this.getAttributeInstance(EntityAttributes.GENERIC_ATTACK_DAMAGE) != null) {
                this.getAttributeInstance(EntityAttributes.GENERIC_ATTACK_DAMAGE).setBaseValue(19.0);
            }
        }
    }

    private void performCrawlerChargeOrTorchPurge(ServerPlayerEntity target) {
        // Arenadaki meşaleleri patlat ve oyuncuya doğru sıçra
        BlockPos origin = this.getBlockPos();
        for (BlockPos pos : BlockPos.iterateOutwards(origin, 20, 6, 20)) {
            BlockState state = this.getWorld().getBlockState(pos);
            if (BaseWreckerGoal.isBreakableLightSource(state)) {
                this.getWorld().breakBlock(pos, false, this);
            }
        }
        Vec3d dir = target.getPos().subtract(this.getPos()).normalize();
        this.setVelocity(dir.x * 1.45, 0.35, dir.z * 1.45);
        this.playSound(ModSounds.STALKER_SLASH, 1.5f, 0.7f);
    }

    private void performPillarAmbushOrClones(ServerPlayerEntity target) {
        if (!(this.getWorld() instanceof ServerWorld serverWorld)) return;

        // 3 adet 1 HP'lik gölge kopyası çağır
        for (int i = 0; i < 3; i++) {
            double angle = (Math.PI * 2.0 * i) / 3.0;
            double sx = target.getX() + Math.cos(angle) * 7.0;
            double sz = target.getZ() + Math.sin(angle) * 7.0;

            GrinningStalkerEntity clone = ModEntities.GRINNING_STALKER.create(serverWorld);
            if (clone != null) {
                clone.refreshPositionAndAngles(sx, target.getY(), sz, 0.0f, 0.0f);
                clone.setVariant(StalkerVariant.HIT_AND_RUN);
                clone.setHealth(1.0f);
                clone.setTarget(target);
                serverWorld.spawnEntity(clone);
            }
        }
        this.playSound(ModSounds.STALKER_VANISH, 1.5f, 0.6f);
    }

    private void startAbyssalGazeCharge(ServerPlayerEntity target) {
        this.dataTracker.set(GAZE_CHARGE_TICKS, 60); // 3 saniye şarj
        this.playSound(ModSounds.APEX_DREAD_GAZE_CHARGE, 2.0f, 1.0f);
        ServerPlayNetworking.send(target, new HallucinationEventS2CPayload(
                HallucinationEventS2CPayload.EventType.ABYSSAL_GAZE_WARNING,
                1.0f
        ));
    }

    private void executeAbyssalGazeBurst() {
        if (!(this.getWorld() instanceof ServerWorld serverWorld)) return;
        for (ServerPlayerEntity player : serverWorld.getPlayers()) {
            if (player.squaredDistanceTo(this) <= 48.0 * 48.0 && player.canSee(this)) {
                Vec3d look = player.getRotationVec(1.0f).normalize();
                Vec3d toBoss = this.getEyePos().subtract(player.getEyePos()).normalize();
                // Oyuncu arkasını dönmediyse (dot > 0.35) ağır hasar ve +200 Dread alır
                if (look.dotProduct(toBoss) > 0.35) {
                    player.damage(serverWorld.getDamageSources().magic(), 15.0f);
                    player.addStatusEffect(new StatusEffectInstance(StatusEffects.DARKNESS, 200, 0));
                    DreadManager.addEventDread(player, 200.0f);
                }
            }
        }
    }

    @Override
    public void onStartedTrackingBy(ServerPlayerEntity player) {
        super.onStartedTrackingBy(player);
        this.bossBar.addPlayer(player);
    }

    @Override
    public void onStoppedTrackingBy(ServerPlayerEntity player) {
        super.onStoppedTrackingBy(player);
        this.bossBar.removePlayer(player);
    }

    @Override
    public void onDeath(DamageSource damageSource) {
        super.onDeath(damageSource);
        if (!this.getWorld().isClient() && this.getWorld() instanceof ServerWorld serverWorld) {
            // 1. Umbral Core ve Charred Bone Block düşür
            this.dropItem(ModItems.UMBRAL_CORE);
            this.dropStack(new ItemStack(ModBlocks.CHARRED_BONE_BLOCK, 16));

            // 2. Arena merkezinde Herobrine Totem Rehberi Kitabını (Tome of the White Eyes) içeren sandığı oluştur
            BlockPos chestPos = this.getBlockPos();
            serverWorld.setBlockState(chestPos, Blocks.CHEST.getDefaultState());
            if (serverWorld.getBlockEntity(chestPos) instanceof ChestBlockEntity chest) {
                chest.setStack(13, new ItemStack(ModItems.TOME_OF_WHITE_EYES));
                chest.setStack(11, new ItemStack(ModItems.UMBRAL_CORE));
                chest.setStack(15, new ItemStack(ModItems.GRINNING_HEART));
            }

            // 3. Katılımcı oyunculara tek seferlik Java Advancement ver
            for (ServerPlayerEntity player : serverWorld.getPlayers()) {
                if (player.squaredDistanceTo(this) <= 64.0 * 64.0) {
                    ModAdvancementCriteria.SLAY_APEX_DREAD.trigger(player);
                }
            }

            serverWorld.spawnParticles(ParticleTypes.SOUL, getX(), getY() + 2.0, getZ(), 80, 1.2, 2.0, 1.2, 0.05);
        }
    }

    @Override
    public boolean cannotDespawn() {
        return true;
    }

    @Override
    public boolean isFireImmune() {
        return true;
    }

    @Override
    protected SoundEvent getAmbientSound() {
        return ModSounds.STALKER_BREATHING;
    }

    @Override
    protected SoundEvent getHurtSound(DamageSource source) {
        return ModSounds.STALKER_BONE_CLICK;
    }

    @Override
    protected SoundEvent getDeathSound() {
        return ModSounds.APEX_DREAD_ROAR;
    }

    @Override
    public void writeCustomDataToNbt(NbtCompound nbt) {
        super.writeCustomDataToNbt(nbt);
        nbt.putInt("BossPhase", getBossPhase());
    }

    @Override
    public void readCustomDataFromNbt(NbtCompound nbt) {
        super.readCustomDataFromNbt(nbt);
        if (nbt.contains("BossPhase")) {
            setBossPhase(nbt.getInt("BossPhase"));
        }
        if (this.hasCustomName()) {
            this.bossBar.setName(this.getDisplayName());
        }
    }

    @Override
    public void registerControllers(AnimatableManager.ControllerRegistrar controllers) {
        controllers.add(new AnimationController<>(this, "apex_controller", 3, this::predicate));
    }

    private PlayState predicate(AnimationState<ApexDreadEntity> state) {
        if (transitionAnimTicks > 0) {
            return state.setAndContinue(getBossPhase() == 2 ? PHASE_TRANSITION_1 : PHASE_TRANSITION_2);
        }
        if (this.handSwinging) {
            return state.setAndContinue(ATTACK_ANIM);
        }
        if (state.isMoving()) {
            return state.setAndContinue(getBossPhase() == 3 ? SPRINT_ANIM : WALK_ANIM);
        }
        return state.setAndContinue(IDLE_ANIM);
    }

    @Override
    public AnimatableInstanceCache getAnimatableInstanceCache() {
        return geoCache;
    }
}
```

## Dosya: `src/main/java/com/studio/dread/entity/boss/HerobrineApparitionEntity.java`
```java
package com.studio.dread.entity.boss;

import com.studio.dread.config.DreadConfig;
import com.studio.dread.core.dread.DreadManager;
import com.studio.dread.network.HallucinationEventS2CPayload;
import com.studio.dread.registry.ModSounds;
import net.fabricmc.fabric.api.networking.v1.ServerPlayNetworking;
import net.minecraft.entity.Entity;
import net.minecraft.entity.EntityType;
import net.minecraft.entity.attribute.DefaultAttributeContainer;
import net.minecraft.entity.attribute.EntityAttributes;
import net.minecraft.entity.damage.DamageSource;
import net.minecraft.entity.data.DataTracker;
import net.minecraft.entity.data.TrackedData;
import net.minecraft.entity.data.TrackedDataHandlerRegistry;
import net.minecraft.entity.effect.StatusEffectInstance;
import net.minecraft.entity.effect.StatusEffects;
import net.minecraft.entity.mob.PathAwareEntity;
import net.minecraft.entity.player.PlayerEntity;
import net.minecraft.particle.ParticleTypes;
import net.minecraft.server.network.ServerPlayerEntity;
import net.minecraft.server.world.ServerWorld;
import net.minecraft.util.math.Vec3d;
import net.minecraft.world.World;
import software.bernie.geckolib.animatable.GeoEntity;
import software.bernie.geckolib.animatable.instance.AnimatableInstanceCache;
import software.bernie.geckolib.animation.AnimatableManager;
import software.bernie.geckolib.animation.AnimationController;
import software.bernie.geckolib.animation.AnimationState;
import software.bernie.geckolib.animation.PlayState;
import software.bernie.geckolib.animation.RawAnimation;
import software.bernie.geckolib.util.GeckoLibUtil;

import java.util.UUID;

/**
 * Herobrine Boss savaşı öncesi psikolojik takipçi:
 * - Saldırmaz, oyuncuyu öldürmez, fiziksel temas kurmaz, hasar vermez.
 * - Uzaktan izler, anlık görünür, kaybolur, jumpscare ve Blindness verir.
 */
public class HerobrineApparitionEntity extends PathAwareEntity implements GeoEntity {
    private static final TrackedData<Boolean> JUMPSCARE_MODE = DataTracker.registerData(HerobrineApparitionEntity.class, TrackedDataHandlerRegistry.BOOLEAN);

    private static final RawAnimation WATCHING_ANIM = RawAnimation.begin().thenLoop("animation.dread.watching");
    private static final RawAnimation JUMPSCARE_ANIM = RawAnimation.begin().thenPlay("animation.dread.jumpscare");

    private final AnimatableInstanceCache geoCache = GeckoLibUtil.createInstanceCache(this);
    private UUID hauntedPlayerUuid;
    private int lifeTicks = 0;
    private int maxLifeTicks = 300;

    public HerobrineApparitionEntity(EntityType<? extends PathAwareEntity> entityType, World world) {
        super(entityType, world);
        this.setInvulnerable(true);
    }

    public static DefaultAttributeContainer.Builder createApparitionAttributes() {
        return PathAwareEntity.createMobAttributes()
                .add(EntityAttributes.GENERIC_MAX_HEALTH, 20.0)
                .add(EntityAttributes.GENERIC_MOVEMENT_SPEED, 0.0);
    }

    @Override
    protected void initDataTracker(DataTracker.Builder builder) {
        super.initDataTracker(builder);
        builder.add(JUMPSCARE_MODE, false);
    }

    public void configureHaunt(ServerPlayerEntity targetPlayer, boolean closeJumpscare) {
        this.hauntedPlayerUuid = targetPlayer.getUuid();
        this.dataTracker.set(JUMPSCARE_MODE, closeJumpscare);
        this.maxLifeTicks = closeJumpscare ? 24 : 360;
    }

    @Override
    public void tick() {
        super.tick();
        if (this.getWorld().isClient()) {
            return;
        }

        lifeTicks++;
        if (lifeTicks >= maxLifeTicks) {
            vanishWithSmoke();
            return;
        }

        PlayerEntity target = (hauntedPlayerUuid != null)
                ? this.getWorld().getPlayerByUuid(hauntedPlayerUuid)
                : this.getWorld().getClosestPlayer(this, 64.0);

        if (target == null || !target.isAlive()) {
            this.discard();
            return;
        }

        this.getLookControl().lookAt(target, 180.0f, 180.0f);

        // Jumpscare modu: Blindness verir, ses çalar, asla hasar vermez ve kaybolur
        if (this.dataTracker.get(JUMPSCARE_MODE)) {
            if (lifeTicks == 2 && target instanceof ServerPlayerEntity serverPlayer) {
                serverPlayer.addStatusEffect(new StatusEffectInstance(StatusEffects.BLINDNESS, 80, 0));
                DreadManager.addEventDread(serverPlayer, DreadConfig.HEROBRINE_SCARE_DREAD_BONUS);
                ServerPlayNetworking.send(serverPlayer, new HallucinationEventS2CPayload(
                        HallucinationEventS2CPayload.EventType.HEROBRINE_JUMPSCARE,
                        1.0f
                ));
                this.playSound(ModSounds.HEROBRINE_JUMPSCARE_STATIC, 1.5f, 1.0f);
            }
            if (lifeTicks >= 18) {
                vanishWithSmoke();
            }
            return;
        }

        // Uzaktan izleme modu: Oyuncu 18 bloktan fazla yaklaşırsa veya doğrudan odaklanırsa kaybolur
        double distSq = this.squaredDistanceTo(target);
        Vec3d look = target.getRotationVec(1.0f).normalize();
        Vec3d toHerobrine = this.getEyePos().subtract(target.getEyePos()).normalize();

        if (distSq < 18.0 * 18.0 || (look.dotProduct(toHerobrine) > 0.94 && lifeTicks > 25)) {
            if (target instanceof ServerPlayerEntity serverPlayer) {
                serverPlayer.addStatusEffect(new StatusEffectInstance(StatusEffects.BLINDNESS, 40, 0));
                ServerPlayNetworking.send(serverPlayer, new HallucinationEventS2CPayload(
                        HallucinationEventS2CPayload.EventType.HEROBRINE_FLASH,
                        0.7f
                ));
            }
            vanishWithSmoke();
        }
    }

    private void vanishWithSmoke() {
        if (this.getWorld() instanceof ServerWorld serverWorld) {
            serverWorld.spawnParticles(ParticleTypes.LARGE_SMOKE, getX(), getY() + 1.0, getZ(), 20, 0.3, 0.6, 0.3, 0.02);
        }
        this.discard();
    }

    @Override
    public boolean isPushable() {
        return false; // Fiziksel temas kurmaz
    }

    @Override
    protected void pushAway(Entity entity) {
        // Fiziksel çarpışma yok
    }

    @Override
    public boolean damage(DamageSource source, float amount) {
        if (!this.getWorld().isClient()) {
            vanishWithSmoke();
        }
        return false;
    }

    @Override
    public boolean tryAttack(Entity target) {
        return false; // Kesinlikle saldırmaz ve hasar vermez
    }

    @Override
    public void registerControllers(AnimatableManager.ControllerRegistrar controllers) {
        controllers.add(new AnimationController<>(this, "apparition_controller", 2, this::predicate));
    }

    private PlayState predicate(AnimationState<HerobrineApparitionEntity> state) {
        if (this.dataTracker.get(JUMPSCARE_MODE)) {
            return state.setAndContinue(JUMPSCARE_ANIM);
        }
        return state.setAndContinue(WATCHING_ANIM);
    }

    @Override
    public AnimatableInstanceCache getAnimatableInstanceCache() {
        return geoCache;
    }
}
```

## Dosya: `src/main/java/com/studio/dread/entity/boss/HerobrineBossEntity.java`
```java
package com.studio.dread.entity.boss;

import com.studio.dread.core.dread.DreadManager;
import com.studio.dread.core.dread.PlayerDreadState;
import com.studio.dread.core.persistence.DreadWorldPersistentState;
import com.studio.dread.entity.ModEntities;
import com.studio.dread.entity.pitchgrin.PitchGrinEntity;
import com.studio.dread.entity.stalker.GrinningStalkerEntity;
import com.studio.dread.entity.stalker.StalkerVariant;
import com.studio.dread.network.HallucinationEventS2CPayload;
import com.studio.dread.registry.ModAdvancementCriteria;
import com.studio.dread.registry.ModBlocks;
import com.studio.dread.registry.ModItems;
import com.studio.dread.registry.ModSounds;
import net.fabricmc.fabric.api.networking.v1.ServerPlayNetworking;
import net.minecraft.block.Block;
import net.minecraft.block.BlockState;
import net.minecraft.block.Blocks;
import net.minecraft.entity.EntityType;
import net.minecraft.entity.LightningEntity;
import net.minecraft.entity.ai.goal.ActiveTargetGoal;
import net.minecraft.entity.ai.goal.LookAtEntityGoal;
import net.minecraft.entity.ai.goal.MeleeAttackGoal;
import net.minecraft.entity.ai.goal.SwimGoal;
import net.minecraft.entity.attribute.DefaultAttributeContainer;
import net.minecraft.entity.attribute.EntityAttributes;
import net.minecraft.entity.boss.BossBar;
import net.minecraft.entity.boss.ServerBossBar;
import net.minecraft.entity.damage.DamageSource;
import net.minecraft.entity.data.DataTracker;
import net.minecraft.entity.data.TrackedData;
import net.minecraft.entity.data.TrackedDataHandlerRegistry;
import net.minecraft.entity.effect.StatusEffectInstance;
import net.minecraft.entity.effect.StatusEffects;
import net.minecraft.entity.mob.HostileEntity;
import net.minecraft.entity.player.PlayerEntity;
import net.minecraft.nbt.NbtCompound;
import net.minecraft.particle.ParticleTypes;
import net.minecraft.registry.Registries;
import net.minecraft.server.network.ServerPlayerEntity;
import net.minecraft.server.world.ServerWorld;
import net.minecraft.sound.SoundEvent;
import net.minecraft.text.Text;
import net.minecraft.util.Formatting;
import net.minecraft.util.Identifier;
import net.minecraft.util.math.BlockPos;
import net.minecraft.util.math.Vec3d;
import net.minecraft.world.World;
import software.bernie.geckolib.animatable.GeoEntity;
import software.bernie.geckolib.animatable.instance.AnimatableInstanceCache;
import software.bernie.geckolib.animation.AnimatableManager;
import software.bernie.geckolib.animation.AnimationController;
import software.bernie.geckolib.animation.AnimationState;
import software.bernie.geckolib.animation.PlayState;
import software.bernie.geckolib.animation.RawAnimation;
import software.bernie.geckolib.util.GeckoLibUtil;

import java.util.Map;

public class HerobrineBossEntity extends HostileEntity implements GeoEntity {
    private static final TrackedData<Integer> PHASE = DataTracker.registerData(HerobrineBossEntity.class, TrackedDataHandlerRegistry.INTEGER);
    private static final TrackedData<Boolean> FALSE_DEATH_STANCE = DataTracker.registerData(HerobrineBossEntity.class, TrackedDataHandlerRegistry.BOOLEAN);

    private static final RawAnimation IDLE_ANIM = RawAnimation.begin().thenLoop("animation.dread.idle");
    private static final RawAnimation WALK_ANIM = RawAnimation.begin().thenLoop("animation.dread.walk");
    private static final RawAnimation SPRINT_ANIM = RawAnimation.begin().thenLoop("animation.dread.sprint");
    private static final RawAnimation ATTACK_ANIM = RawAnimation.begin().thenPlay("animation.dread.attack");
    private static final RawAnimation PHASE_1_TO_2_ANIM = RawAnimation.begin().thenPlay("animation.dread.boss_phase_1_to_2");
    private static final RawAnimation PHASE_2_TO_3_ANIM = RawAnimation.begin().thenPlay("animation.dread.boss_phase_2_to_3");
    private static final RawAnimation OBSERVE_KNEEL_ANIM = RawAnimation.begin().thenPlayAndHold("animation.dread.observe");

    private final ServerBossBar bossBar = new ServerBossBar(
            Text.translatable("entity.dread.herobrine_boss").formatted(Formatting.WHITE, Formatting.BOLD),
            BossBar.Color.WHITE,
            BossBar.Style.NOTCHED_20
    );

    private final AnimatableInstanceCache geoCache = GeckoLibUtil.createInstanceCache(this);
    private BlockPos arenaOrigin = BlockPos.ORIGIN;
    private int abilityCooldown = 80;
    private int phaseTransitionTicks = 0;
    private int falseDeathWindowTicks = 0;
    private boolean triggeredFalseDeath = false;

    public HerobrineBossEntity(EntityType<? extends HostileEntity> entityType, World world) {
        super(entityType, world);
        this.experiencePoints = 1000;
    }

    public static DefaultAttributeContainer.Builder createHerobrineBossAttributes() {
        return HostileEntity.createHostileAttributes()
                .add(EntityAttributes.GENERIC_MAX_HEALTH, 800.0)
                .add(EntityAttributes.GENERIC_MOVEMENT_SPEED, 0.36)
                .add(EntityAttributes.GENERIC_ATTACK_DAMAGE, 15.0)
                .add(EntityAttributes.GENERIC_ARMOR, 14.0)
                .add(EntityAttributes.GENERIC_FOLLOW_RANGE, 80.0)
                .add(EntityAttributes.GENERIC_KNOCKBACK_RESISTANCE, 1.0)
                .add(EntityAttributes.GENERIC_STEP_HEIGHT, 1.5);
    }

    @Override
    protected void initDataTracker(DataTracker.Builder builder) {
        super.initDataTracker(builder);
        builder.add(PHASE, 1);
        builder.add(FALSE_DEATH_STANCE, false);
    }

    @Override
    protected void initGoals() {
        this.goalSelector.add(0, new SwimGoal(this));
        this.goalSelector.add(2, new MeleeAttackGoal(this, 1.4, false));
        this.goalSelector.add(6, new LookAtEntityGoal(this, PlayerEntity.class, 64.0f));
        this.targetSelector.add(1, new ActiveTargetGoal<>(this, PlayerEntity.class, true));
    }

    public void initializeArena(BlockPos totemPos) {
        this.arenaOrigin = totemPos.toImmutable();
    }

    public int getPhase() {
        return this.dataTracker.get(PHASE);
    }

    public void setPhase(int phase) {
        this.dataTracker.set(PHASE, phase);
    }

    @Override
    public void tick() {
        super.tick();
        if (this.getWorld().isClient() || !(this.getWorld() instanceof ServerWorld serverWorld)) {
            return;
        }

        this.bossBar.setPercent(this.getHealth() / this.getMaxHealth());
        checkPhaseTransitions(serverWorld);

        if (phaseTransitionTicks > 0) {
            phaseTransitionTicks--;
            this.getNavigation().stop();
            return;
        }

        if (falseDeathWindowTicks > 0) {
            falseDeathWindowTicks--;
            this.getNavigation().stop();
            if (falseDeathWindowTicks == 1) {
                this.dataTracker.set(FALSE_DEATH_STANCE, false);
            }
            return;
        }

        if (--abilityCooldown <= 0 && this.getTarget() instanceof ServerPlayerEntity target) {
            int phase = getPhase();
            if (phase == 1) {
                executePhase1Abilities(serverWorld, target);
                abilityCooldown = 90;
            } else if (phase == 2) {
                executePhase2Abilities(serverWorld, target);
                abilityCooldown = 80;
            } else {
                executePhase3Abilities(serverWorld, target);
                abilityCooldown = 65;
            }
        }
    }

    private void checkPhaseTransitions(ServerWorld serverWorld) {
        float ratio = this.getHealth() / this.getMaxHealth();
        int phase = getPhase();

        if (phase == 1 && ratio <= 0.65f) {
            setPhase(2);
            phaseTransitionTicks = 55;
            this.bossBar.setColor(BossBar.Color.RED);
            this.playSound(ModSounds.MUSIC_BOSS_HEROBRINE_P2, 2.0f, 1.0f);
            corruptArenaTerrain(serverWorld);
        } else if (phase == 2 && ratio <= 0.275f) {
            setPhase(3);
            phaseTransitionTicks = 55;
            this.bossBar.setColor(BossBar.Color.PURPLE);
            this.playSound(ModSounds.MUSIC_BOSS_HEROBRINE_P3, 2.0f, 1.0f);

            // Faz 3: Arena içindeki oyuncuların Dread barını 999/1000 seviyesine kilitle
            DreadWorldPersistentState worldState = DreadWorldPersistentState.get(serverWorld.getServer());
            for (ServerPlayerEntity player : serverWorld.getPlayers()) {
                if (player.squaredDistanceTo(this) <= 80.0 * 80.0) {
                    PlayerDreadState pState = worldState.getPlayerState(player.getUuid());
                    pState.setLockedAtMaxByBoss(true);
                    DreadManager.forceSync(player, pState);
                }
            }
        }

        // Faz 3 %10 Can: Sahte Ölüm (False Death & Jumpscare Counter)
        if (phase == 3 && !triggeredFalseDeath && ratio <= 0.10f) {
            triggeredFalseDeath = true;
            falseDeathWindowTicks = 60; // 3 saniye diz çökme tuzağı
            this.dataTracker.set(FALSE_DEATH_STANCE, true);
        }
    }

    /**
     * Faz 1: Void Step, Yıldırım Çağrısı ve 6'lı İllüzyon Çemberi (Hallucination Ring)
     */
    private void executePhase1Abilities(ServerWorld world, ServerPlayerEntity target) {
        int roll = world.getRandom().nextInt(3);
        if (roll == 0) {
            // Void Step: Oyuncunun arkasına ışınlan ve vur
            Vec3d back = target.getPos().subtract(target.getRotationVec(1.0f).normalize().multiply(2.5));
            this.teleport(back.x, target.getY(), back.z, true);
            this.playSound(ModSounds.STALKER_VANISH, 1.4f, 1.2f);
        } else if (roll == 1) {
            // Lightning Call
            for (int i = 0; i < 4; i++) {
                BlockPos strikePos = target.getBlockPos().add(
                        world.getRandom().nextBetween(-5, 5),
                        0,
                        world.getRandom().nextBetween(-5, 5)
                );
                LightningEntity bolt = EntityType.LIGHTNING_BOLT.create(world);
                if (bolt != null) {
                    bolt.refreshPositionAfterTeleport(strikePos.getX() + 0.5, strikePos.getY(), strikePos.getZ() + 0.5);
                    world.spawnEntity(bolt);
                }
            }
        } else {
            // Hallucination Ring: 5 adet 0 hasarlı Herobrine İllüzyonu + 1 gerçek Herobrine
            for (int i = 0; i < 5; i++) {
                double angle = (Math.PI * 2.0 * i) / 5.0;
                double ix = target.getX() + Math.cos(angle) * 7.0;
                double iz = target.getZ() + Math.sin(angle) * 7.0;
                HerobrineApparitionEntity illusion = ModEntities.HEROBRINE_APPARITION.create(world);
                if (illusion != null) {
                    illusion.refreshPositionAndAngles(ix, target.getY(), iz, 0.0f, 0.0f);
                    illusion.configureHaunt(target, false);
                    world.spawnEntity(illusion);
                }
            }
            double realAngle = world.getRandom().nextDouble() * Math.PI * 2.0;
            this.teleport(target.getX() + Math.cos(realAngle) * 7.0, target.getY(), target.getZ() + Math.sin(realAngle) * 7.0, true);
        }
    }

    /**
     * Faz 2: Dünya Manipülasyonu (Terrain Shatter) ve Yardımcı Yaratık Çağırma (Dread Summoning)
     */
    private void executePhase2Abilities(ServerWorld world, ServerPlayerEntity target) {
        if (world.getRandom().nextBoolean()) {
            // Yardımcı Yaratık Çağırma: 1 Pitch Grin + 1 Hit & Run Stalker
            PitchGrinEntity minion1 = ModEntities.PITCH_GRIN.create(world);
            if (minion1 != null) {
                minion1.refreshPositionAndAngles(this.getX() + 2.5, this.getY(), this.getZ(), 0.0f, 0.0f);
                minion1.setHealth(40.0f);
                minion1.setTarget(target);
                world.spawnEntity(minion1);
            }

            GrinningStalkerEntity minion2 = ModEntities.GRINNING_STALKER.create(world);
            if (minion2 != null) {
                minion2.refreshPositionAndAngles(this.getX() - 2.5, this.getY(), this.getZ(), 0.0f, 0.0f);
                minion2.setVariant(StalkerVariant.HIT_AND_RUN);
                minion2.setHealth(45.0f);
                minion2.setTarget(target);
                world.spawnEntity(minion2);
            }
            this.playSound(ModSounds.APEX_DREAD_ROAR, 1.5f, 1.1f);
        } else {
            // Graveyard Pillars & Ground Shatter
            DreadWorldPersistentState state = DreadWorldPersistentState.get(world.getServer());
            BlockPos tPos = target.getBlockPos();
            for (int dx = -2; dx <= 2; dx += 4) {
                for (int dz = -2; dz <= 2; dz += 4) {
                    for (int dy = 0; dy <= 2; dy++) {
                        BlockPos pillarPos = tPos.add(dx, dy, dz);
                        BlockState existing = world.getBlockState(pillarPos);
                        if (existing.isAir()) {
                            state.recordArenaBlockBeforeCorruption(pillarPos, Registries.BLOCK.getId(existing.getBlock()).toString());
                            world.setBlockState(pillarPos, ModBlocks.CHARRED_BONE_BLOCK.getDefaultState());
                        }
                    }
                }
            }
        }
    }

    /**
     * Faz 3: Omnislash Barrage & Skyfall Meteor
     */
    private void executePhase3Abilities(ServerWorld world, ServerPlayerEntity target) {
        Vec3d offset = new Vec3d(
                (world.getRandom().nextDouble() - 0.5) * 6.0,
                0.5,
                (world.getRandom().nextDouble() - 0.5) * 6.0
        );
        this.teleport(target.getX() + offset.x, target.getY() + offset.y, target.getZ() + offset.z, true);
        this.tryAttack(target);

        LightningEntity bolt = EntityType.LIGHTNING_BOLT.create(world);
        if (bolt != null) {
            bolt.refreshPositionAfterTeleport(target.getX(), target.getY(), target.getZ());
            world.spawnEntity(bolt);
        }
    }

    private void corruptArenaTerrain(ServerWorld world) {
        DreadWorldPersistentState state = DreadWorldPersistentState.get(world.getServer());
        BlockPos center = this.arenaOrigin.equals(BlockPos.ORIGIN) ? this.getBlockPos() : this.arenaOrigin;

        for (BlockPos pos : BlockPos.iterateOutwards(center, 18, 4, 18)) {
            BlockState current = world.getBlockState(pos);
            if (current.isOf(Blocks.GRASS_BLOCK) && world.getRandom().nextFloat() < 0.35f) {
                state.recordArenaBlockBeforeCorruption(pos, Registries.BLOCK.getId(current.getBlock()).toString());
                world.setBlockState(pos, ModBlocks.ASHEN_SOIL.getDefaultState(), Block.NOTIFY_LISTENERS);
            } else if (current.isOf(Blocks.TORCH)) {
                state.recordArenaBlockBeforeCorruption(pos, Registries.BLOCK.getId(current.getBlock()).toString());
                world.setBlockState(pos, Blocks.REDSTONE_TORCH.getDefaultState(), Block.NOTIFY_LISTENERS);
            }
        }
    }

    private void restoreArenaTerrain(ServerWorld world) {
        DreadWorldPersistentState state = DreadWorldPersistentState.get(world.getServer());
        for (Map.Entry<BlockPos, String> entry : state.getCorruptedArenaBlocks().entrySet()) {
            Block originalBlock = Registries.BLOCK.get(Identifier.of(entry.getValue()));
            world.setBlockState(entry.getKey(), originalBlock.getDefaultState(), Block.NOTIFY_ALL);
        }
        state.getCorruptedArenaBlocks().clear();
        state.markDirty();
    }

    @Override
    public boolean damage(DamageSource source, float amount) {
        // Sahte ölüm tuzağındayken oyuncu vurursa Counter Jumpscare tetiklenir
        if (this.dataTracker.get(FALSE_DEATH_STANCE) && source.getAttacker() instanceof ServerPlayerEntity player) {
            this.dataTracker.set(FALSE_DEATH_STANCE, false);
            this.falseDeathWindowTicks = 0;
            player.addStatusEffect(new StatusEffectInstance(StatusEffects.DARKNESS, 120, 0));
            ServerPlayNetworking.send(player, new HallucinationEventS2CPayload(
                    HallucinationEventS2CPayload.EventType.HEROBRINE_JUMPSCARE,
                    1.0f
            ));
            player.damage(this.getWorld().getDamageSources().mobAttack(this), 14.0f);
            return false;
        }
        return super.damage(source, amount);
    }

    @Override
    public void onStartedTrackingBy(ServerPlayerEntity player) {
        super.onStartedTrackingBy(player);
        this.bossBar.addPlayer(player);
    }

    @Override
    public void onStoppedTrackingBy(ServerPlayerEntity player) {
        super.onStoppedTrackingBy(player);
        this.bossBar.removePlayer(player);
    }

    @Override
    public void onDeath(DamageSource damageSource) {
        super.onDeath(damageSource);
        if (!this.getWorld().isClient() && this.getWorld() instanceof ServerWorld serverWorld) {
            // 1. Efsanevi son oyun ödülü: Eclipse of the Hollow düşür
            this.dropItem(ModItems.ECLIPSE_OF_THE_HOLLOW);

            // 2. Arena bloklarını eski haline döndür ve fırtınayı bitir
            restoreArenaTerrain(serverWorld);
            serverWorld.setWeather(12000, 0, false, false);
            serverWorld.setTimeOfDay(1000L);

            // 3. Oyuncuların Dread kilidini aç, Dread'i sıfırla ve başarımı ver
            DreadWorldPersistentState worldState = DreadWorldPersistentState.get(serverWorld.getServer());
            for (ServerPlayerEntity player : serverWorld.getPlayers()) {
                if (player.squaredDistanceTo(this) <= 96.0 * 96.0) {
                    PlayerDreadState pState = worldState.getPlayerState(player.getUuid());
                    pState.setLockedAtMaxByBoss(false);
                    pState.setDreadScore(0.0f);
                    DreadManager.forceSync(player, pState);
                    ModAdvancementCriteria.SLAY_HEROBRINE_BOSS.trigger(player);
                }
            }

            serverWorld.spawnParticles(ParticleTypes.END_ROD, getX(), getY() + 1.5, getZ(), 120, 1.5, 2.0, 1.5, 0.15);
        }
    }

    @Override
    public boolean cannotDespawn() {
        return true;
    }

    @Override
    public boolean isFireImmune() {
        return true;
    }

    @Override
    protected SoundEvent getHurtSound(DamageSource source) {
        return ModSounds.HEROBRINE_JUMPSCARE_STATIC;
    }

    @Override
    protected SoundEvent getDeathSound() {
        return ModSounds.HEROBRINE_TOTEM_AWAKEN;
    }

    @Override
    public void writeCustomDataToNbt(NbtCompound nbt) {
        super.writeCustomDataToNbt(nbt);
        nbt.putInt("HerobrinePhase", getPhase());
        nbt.putLong("ArenaOrigin", arenaOrigin.asLong());
    }

    @Override
    public void readCustomDataFromNbt(NbtCompound nbt) {
        super.readCustomDataFromNbt(nbt);
        if (nbt.contains("HerobrinePhase")) {
            setPhase(nbt.getInt("HerobrinePhase"));
        }
        if (nbt.contains("ArenaOrigin")) {
            this.arenaOrigin = BlockPos.fromLong(nbt.getLong("ArenaOrigin"));
        }
    }

    @Override
    public void registerControllers(AnimatableManager.ControllerRegistrar controllers) {
        controllers.add(new AnimationController<>(this, "herobrine_boss_controller", 2, this::predicate));
    }

    private PlayState predicate(AnimationState<HerobrineBossEntity> state) {
        if (this.dataTracker.get(FALSE_DEATH_STANCE)) {
            return state.setAndContinue(OBSERVE_KNEEL_ANIM);
        }
        if (phaseTransitionTicks > 0) {
            return state.setAndContinue(getPhase() == 2 ? PHASE_1_TO_2_ANIM : PHASE_2_TO_3_ANIM);
        }
        if (this.handSwinging) {
            return state.setAndContinue(ATTACK_ANIM);
        }
        if (state.isMoving()) {
            return state.setAndContinue(getPhase() == 3 ? SPRINT_ANIM : WALK_ANIM);
        }
        return state.setAndContinue(IDLE_ANIM);
    }

    @Override
    public AnimatableInstanceCache getAnimatableInstanceCache() {
        return geoCache;
    }
}
```

## Dosya: `src/main/java/com/studio/dread/entity/client/ApexDreadModel.java`
```java
package com.studio.dread.entity.client;

import com.studio.dread.config.DreadConfig;
import com.studio.dread.entity.boss.ApexDreadEntity;
import net.minecraft.util.Identifier;
import software.bernie.geckolib.model.GeoModel;

public class ApexDreadModel extends GeoModel<ApexDreadEntity> {
    private static final Identifier MODEL = Identifier.of(DreadConfig.MOD_ID, "geo/apex_dread.geo.json");
    private static final Identifier TEXTURE = Identifier.of(DreadConfig.MOD_ID, "textures/entity/apex_dread.png");
    private static final Identifier ANIMATION = Identifier.of(DreadConfig.MOD_ID, "animations/dread_entities.animation.json");

    @Override
    public Identifier getModelResource(ApexDreadEntity animatable) {
        return MODEL;
    }

    @Override
    public Identifier getTextureResource(ApexDreadEntity animatable) {
        return TEXTURE;
    }

    @Override
    public Identifier getAnimationResource(ApexDreadEntity animatable) {
        return ANIMATION;
    }
}
```

## Dosya: `src/main/java/com/studio/dread/entity/client/ApexDreadRenderer.java`
```java
package com.studio.dread.entity.client;

import com.studio.dread.entity.boss.ApexDreadEntity;
import net.minecraft.client.render.entity.EntityRendererFactory;
import software.bernie.geckolib.renderer.GeoEntityRenderer;
import software.bernie.geckolib.renderer.layer.AutoGlowingGeoLayer;

public class ApexDreadRenderer extends GeoEntityRenderer<ApexDreadEntity> {
    public ApexDreadRenderer(EntityRendererFactory.Context renderManager) {
        super(renderManager, new ApexDreadModel());
        this.shadowRadius = 1.4f;
        addRenderLayer(new AutoGlowingGeoLayer<>(this));
    }
}
```

## Dosya: `src/main/java/com/studio/dread/entity/client/GrinningStalkerModel.java`
```java
package com.studio.dread.entity.client;

import com.studio.dread.config.DreadConfig;
import com.studio.dread.entity.stalker.GrinningStalkerEntity;
import net.minecraft.util.Identifier;
import net.minecraft.util.math.MathHelper;
import software.bernie.geckolib.animation.AnimationState;
import software.bernie.geckolib.cache.object.GeoBone;
import software.bernie.geckolib.constant.DataTickets;
import software.bernie.geckolib.model.GeoModel;
import software.bernie.geckolib.model.data.EntityModelData;

public class GrinningStalkerModel extends GeoModel<GrinningStalkerEntity> {
    private static final Identifier MODEL = Identifier.of(DreadConfig.MOD_ID, "geo/grinning_stalker.geo.json");
    private static final Identifier TEXTURE = Identifier.of(DreadConfig.MOD_ID, "textures/entity/grinning_stalker.png");
    private static final Identifier ANIMATION = Identifier.of(DreadConfig.MOD_ID, "animations/dread_entities.animation.json");

    @Override
    public Identifier getModelResource(GrinningStalkerEntity animatable) {
        return MODEL;
    }

    @Override
    public Identifier getTextureResource(GrinningStalkerEntity animatable) {
        return TEXTURE;
    }

    @Override
    public Identifier getAnimationResource(GrinningStalkerEntity animatable) {
        return ANIMATION;
    }

    @Override
    public void setCustomAnimations(GrinningStalkerEntity animatable, long instanceId, AnimationState<GrinningStalkerEntity> animationState) {
        GeoBone head = getAnimationProcessor().getBone("head");
        if (head != null) {
            EntityModelData entityData = animationState.getData(DataTickets.ENTITY_MODEL_DATA);
            if (entityData != null) {
                head.setRotX(entityData.headPitch() * MathHelper.RADIANS_PER_DEGREE);
                head.setRotY(entityData.netHeadYaw() * MathHelper.RADIANS_PER_DEGREE);
            }
        }
    }
}
```

## Dosya: `src/main/java/com/studio/dread/entity/client/GrinningStalkerRenderer.java`
```java
package com.studio.dread.entity.client;

import com.studio.dread.entity.stalker.GrinningStalkerEntity;
import net.minecraft.client.render.entity.EntityRendererFactory;
import software.bernie.geckolib.renderer.GeoEntityRenderer;
import software.bernie.geckolib.renderer.layer.AutoGlowingGeoLayer;

public class GrinningStalkerRenderer extends GeoEntityRenderer<GrinningStalkerEntity> {
    public GrinningStalkerRenderer(EntityRendererFactory.Context renderManager) {
        super(renderManager, new GrinningStalkerModel());
        this.shadowRadius = 0.75f;
        addRenderLayer(new AutoGlowingGeoLayer<>(this));
    }
}
```

## Dosya: `src/main/java/com/studio/dread/entity/client/HerobrineModel.java`
```java
package com.studio.dread.entity.client;

import com.studio.dread.config.DreadConfig;
import com.studio.dread.entity.boss.HerobrineBossEntity;
import net.minecraft.entity.Entity;
import net.minecraft.util.Identifier;
import software.bernie.geckolib.animatable.GeoEntity;
import software.bernie.geckolib.model.GeoModel;

public class HerobrineModel<T extends Entity & GeoEntity> extends GeoModel<T> {
    private static final Identifier MODEL = Identifier.of(DreadConfig.MOD_ID, "geo/herobrine.geo.json");
    private static final Identifier TEXTURE_NORMAL = Identifier.of(DreadConfig.MOD_ID, "textures/entity/herobrine.png");
    private static final Identifier TEXTURE_GLITCH = Identifier.of(DreadConfig.MOD_ID, "textures/entity/herobrine_glitch.png");
    private static final Identifier ANIMATION = Identifier.of(DreadConfig.MOD_ID, "animations/dread_entities.animation.json");

    @Override
    public Identifier getModelResource(T animatable) {
        return MODEL;
    }

    @Override
    public Identifier getTextureResource(T animatable) {
        if (animatable instanceof HerobrineBossEntity boss && boss.getPhase() == 3) {
            return TEXTURE_GLITCH;
        }
        return TEXTURE_NORMAL;
    }

    @Override
    public Identifier getAnimationResource(T animatable) {
        return ANIMATION;
    }
}
```

## Dosya: `src/main/java/com/studio/dread/entity/client/HerobrineRenderer.java`
```java
package com.studio.dread.entity.client;

import net.minecraft.client.render.entity.EntityRendererFactory;
import net.minecraft.entity.mob.MobEntity;
import software.bernie.geckolib.animatable.GeoEntity;
import software.bernie.geckolib.renderer.GeoEntityRenderer;
import software.bernie.geckolib.renderer.layer.AutoGlowingGeoLayer;

public class HerobrineRenderer<T extends MobEntity & GeoEntity> extends GeoEntityRenderer<T> {
    public HerobrineRenderer(EntityRendererFactory.Context renderManager) {
        super(renderManager, new HerobrineModel<>());
        this.shadowRadius = 0.5f;
        addRenderLayer(new AutoGlowingGeoLayer<>(this));
    }
}
```

## Dosya: `src/main/java/com/studio/dread/entity/client/PitchGrinModel.java`
```java
package com.studio.dread.entity.client;

import com.studio.dread.config.DreadConfig;
import com.studio.dread.entity.pitchgrin.PitchGrinEntity;
import net.minecraft.util.Identifier;
import net.minecraft.util.math.MathHelper;
import software.bernie.geckolib.animation.AnimationState;
import software.bernie.geckolib.cache.object.GeoBone;
import software.bernie.geckolib.constant.DataTickets;
import software.bernie.geckolib.model.GeoModel;
import software.bernie.geckolib.model.data.EntityModelData;

public class PitchGrinModel extends GeoModel<PitchGrinEntity> {
    private static final Identifier MODEL = Identifier.of(DreadConfig.MOD_ID, "geo/pitch_grin.geo.json");
    private static final Identifier TEXTURE = Identifier.of(DreadConfig.MOD_ID, "textures/entity/pitch_grin.png");
    private static final Identifier ANIMATION = Identifier.of(DreadConfig.MOD_ID, "animations/dread_entities.animation.json");

    @Override
    public Identifier getModelResource(PitchGrinEntity animatable) {
        return MODEL;
    }

    @Override
    public Identifier getTextureResource(PitchGrinEntity animatable) {
        return TEXTURE;
    }

    @Override
    public Identifier getAnimationResource(PitchGrinEntity animatable) {
        return ANIMATION;
    }

    @Override
    public void setCustomAnimations(PitchGrinEntity animatable, long instanceId, AnimationState<PitchGrinEntity> animationState) {
        GeoBone head = getAnimationProcessor().getBone("head");
        if (head != null) {
            EntityModelData entityData = animationState.getData(DataTickets.ENTITY_MODEL_DATA);
            if (entityData != null) {
                head.setRotX(entityData.headPitch() * MathHelper.RADIANS_PER_DEGREE);
                head.setRotY(entityData.netHeadYaw() * MathHelper.RADIANS_PER_DEGREE);
            }
        }
    }
}
```

## Dosya: `src/main/java/com/studio/dread/entity/client/PitchGrinRenderer.java`
```java
package com.studio.dread.entity.client;

import com.studio.dread.entity.pitchgrin.PitchGrinEntity;
import net.minecraft.client.render.entity.EntityRendererFactory;
import software.bernie.geckolib.renderer.GeoEntityRenderer;
import software.bernie.geckolib.renderer.layer.AutoGlowingGeoLayer;

public class PitchGrinRenderer extends GeoEntityRenderer<PitchGrinEntity> {
    public PitchGrinRenderer(EntityRendererFactory.Context renderManager) {
        super(renderManager, new PitchGrinModel());
        this.shadowRadius = 0.0f; // Işık emen gölgesiz varlık; yalnızca emissive sırıtışı parlar
        addRenderLayer(new AutoGlowingGeoLayer<>(this));
    }
}
```

## Dosya: `src/main/java/com/studio/dread/entity/pitchgrin/PitchGrinEntity.java`
```java
package com.studio.dread.entity.pitchgrin;

import com.studio.dread.config.DreadConfig;
import com.studio.dread.core.dread.DreadManager;
import com.studio.dread.entity.pitchgrin.ai.CeilingLurkGoal;
import com.studio.dread.entity.pitchgrin.ai.TunnelHuntGoal;
import com.studio.dread.entity.stalker.ai.BaseWreckerGoal;
import com.studio.dread.registry.ModSounds;
import net.minecraft.block.BlockState;
import net.minecraft.entity.Entity;
import net.minecraft.entity.EntityDimensions;
import net.minecraft.entity.EntityPose;
import net.minecraft.entity.EntityType;
import net.minecraft.entity.ai.goal.LookAroundGoal;
import net.minecraft.entity.ai.goal.LookAtEntityGoal;
import net.minecraft.entity.ai.goal.SwimGoal;
import net.minecraft.entity.attribute.DefaultAttributeContainer;
import net.minecraft.entity.attribute.EntityAttributes;
import net.minecraft.entity.damage.DamageSource;
import net.minecraft.entity.data.DataTracker;
import net.minecraft.entity.data.TrackedData;
import net.minecraft.entity.data.TrackedDataHandlerRegistry;
import net.minecraft.entity.effect.StatusEffectInstance;
import net.minecraft.entity.effect.StatusEffects;
import net.minecraft.entity.mob.HostileEntity;
import net.minecraft.entity.player.PlayerEntity;
import net.minecraft.particle.ParticleTypes;
import net.minecraft.server.network.ServerPlayerEntity;
import net.minecraft.sound.SoundCategory;
import net.minecraft.sound.SoundEvent;
import net.minecraft.util.math.BlockPos;
import net.minecraft.world.World;
import software.bernie.geckolib.animatable.GeoEntity;
import software.bernie.geckolib.animatable.instance.AnimatableInstanceCache;
import software.bernie.geckolib.animation.AnimatableManager;
import software.bernie.geckolib.animation.AnimationController;
import software.bernie.geckolib.animation.AnimationState;
import software.bernie.geckolib.animation.PlayState;
import software.bernie.geckolib.animation.RawAnimation;
import software.bernie.geckolib.util.GeckoLibUtil;

public class PitchGrinEntity extends HostileEntity implements GeoEntity {
    private static final TrackedData<Boolean> CEILING_CLINGING = DataTracker.registerData(PitchGrinEntity.class, TrackedDataHandlerRegistry.BOOLEAN);
    private static final TrackedData<Boolean> CRAWLING_MODE = DataTracker.registerData(PitchGrinEntity.class, TrackedDataHandlerRegistry.BOOLEAN);

    private static final EntityDimensions CRAWL_DIM = EntityDimensions.changing(1.3f, 1.45f).withEyeHeight(1.0f);
    private static final EntityDimensions UPRIGHT_DIM = EntityDimensions.changing(1.4f, 3.5f).withEyeHeight(3.1f);

    private static final RawAnimation CRAWL_ANIM = RawAnimation.begin().thenLoop("animation.dread.crawl");
    private static final RawAnimation CLIMB_ANIM = RawAnimation.begin().thenLoop("animation.dread.climb");
    private static final RawAnimation SPRINT_ANIM = RawAnimation.begin().thenLoop("animation.dread.sprint");
    private static final RawAnimation ATTACK_ANIM = RawAnimation.begin().thenPlay("animation.dread.attack");
    private static final RawAnimation IDLE_ANIM = RawAnimation.begin().thenLoop("animation.dread.idle");

    private final AnimatableInstanceCache geoCache = GeckoLibUtil.createInstanceCache(this);

    public PitchGrinEntity(EntityType<? extends HostileEntity> entityType, World world) {
        super(entityType, world);
        this.experiencePoints = 20;
    }

    public static DefaultAttributeContainer.Builder createPitchGrinAttributes() {
        return HostileEntity.createHostileAttributes()
                .add(EntityAttributes.GENERIC_MAX_HEALTH, 110.0)
                .add(EntityAttributes.GENERIC_MOVEMENT_SPEED, 0.33)
                .add(EntityAttributes.GENERIC_ATTACK_DAMAGE, 10.0)
                .add(EntityAttributes.GENERIC_FOLLOW_RANGE, 48.0)
                .add(EntityAttributes.GENERIC_KNOCKBACK_RESISTANCE, 0.6)
                .add(EntityAttributes.GENERIC_STEP_HEIGHT, 1.25);
    }

    @Override
    protected void initDataTracker(DataTracker.Builder builder) {
        super.initDataTracker(builder);
        builder.add(CEILING_CLINGING, false);
        builder.add(CRAWLING_MODE, true);
    }

    @Override
    protected void initGoals() {
        this.goalSelector.add(0, new SwimGoal(this));
        this.goalSelector.add(1, new CeilingLurkGoal(this));
        this.goalSelector.add(2, new TunnelHuntGoal(this));
        this.goalSelector.add(7, new LookAtEntityGoal(this, PlayerEntity.class, 32.0f));
        this.goalSelector.add(8, new LookAroundGoal(this));
    }

    public boolean isCeilingClinging() {
        return this.dataTracker.get(CEILING_CLINGING);
    }

    public void setCeilingClinging(boolean clinging) {
        this.dataTracker.set(CEILING_CLINGING, clinging);
        this.setNoGravity(clinging);
    }

    public boolean isCrawlingMode() {
        return this.dataTracker.get(CRAWLING_MODE);
    }

    public void setCrawlingMode(boolean crawling) {
        if (isCrawlingMode() != crawling) {
            this.dataTracker.set(CRAWLING_MODE, crawling);
            this.calculateDimensions();
        }
    }

    @Override
    protected EntityDimensions getBaseDimensions(EntityPose pose) {
        return isCrawlingMode() ? CRAWL_DIM : UPRIGHT_DIM;
    }

    @Override
    public void tick() {
        super.tick();
        if (this.getWorld().isClient()) {
            // Işık emen siyah duman partikülleri
            for (int i = 0; i < 2; i++) {
                this.getWorld().addParticle(
                        ParticleTypes.SMOKE,
                        this.getParticleX(0.6),
                        this.getRandomBodyY(),
                        this.getParticleZ(0.6),
                        0.0, 0.01, 0.0
                );
            }
        } else {
            // Güneş ışığına çıkarsa veya Y >= 55 üstüne çıkarsa karanlığa gömülüp kaybolur
            if (this.getBlockY() > DreadConfig.PITCH_GRIN_MAX_Y + 4 || this.getWorld().isSkyVisible(this.getBlockPos())) {
                this.playSound(ModSounds.STALKER_VANISH, 1.0f, 0.7f);
                this.discard();
            }
        }
    }

    @Override
    public boolean tryAttack(Entity target) {
        boolean hit = super.tryAttack(target);
        if (hit && target instanceof PlayerEntity player) {
            // Darkness Attack: Darkness verir ve en yakın 2 meşaleyi söndürür
            player.addStatusEffect(new StatusEffectInstance(StatusEffects.DARKNESS, 200, 0));
            if (player instanceof ServerPlayerEntity serverPlayer) {
                DreadManager.addEventDread(serverPlayer, DreadConfig.PITCH_GRIN_ATTACK_DREAD_BONUS);
            }
            extinguishClosestTorches(2);
            this.playSound(ModSounds.PITCH_GRIN_SHRIEK_BITE, 1.3f, 0.9f);
        }
        return hit;
    }

    private void extinguishClosestTorches(int maxCount) {
        int broken = 0;
        BlockPos origin = this.getBlockPos();
        for (BlockPos pos : BlockPos.iterateOutwards(origin, 10, 5, 10)) {
            BlockState state = this.getWorld().getBlockState(pos);
            if (BaseWreckerGoal.isBreakableLightSource(state)) {
                this.getWorld().breakBlock(pos, true, this);
                this.getWorld().playSound(null, pos, ModSounds.AMBIENT_FAKE_TORCH_HISS, SoundCategory.HOSTILE, 0.9f, 0.6f);
                if (++broken >= maxCount) break;
            }
        }
    }

    @Override
    protected SoundEvent getAmbientSound() {
        return ModSounds.STALKER_BREATHING;
    }

    @Override
    protected SoundEvent getHurtSound(DamageSource source) {
        return ModSounds.STALKER_BONE_CLICK;
    }

    @Override
    protected SoundEvent getDeathSound() {
        return ModSounds.STALKER_VANISH;
    }

    @Override
    protected void playStepSound(BlockPos pos, BlockState state) {
        this.playSound(ModSounds.PITCH_GRIN_CRAWL, 0.5f, 0.85f + this.random.nextFloat() * 0.3f);
    }

    @Override
    public void registerControllers(AnimatableManager.ControllerRegistrar controllers) {
        controllers.add(new AnimationController<>(this, "pitch_grin_controller", 2, this::predicate));
    }

    private PlayState predicate(AnimationState<PitchGrinEntity> state) {
        if (this.handSwinging) {
            return state.setAndContinue(ATTACK_ANIM);
        }
        if (isCeilingClinging()) {
            return state.setAndContinue(CLIMB_ANIM);
        }
        if (state.isMoving()) {
            return state.setAndContinue(this.isSprinting() ? SPRINT_ANIM : CRAWL_ANIM);
        }
        return state.setAndContinue(IDLE_ANIM);
    }

    @Override
    public AnimatableInstanceCache getAnimatableInstanceCache() {
        return geoCache;
    }
}
```

## Dosya: `src/main/java/com/studio/dread/entity/pitchgrin/ai/CeilingLurkGoal.java`
```java
package com.studio.dread.entity.pitchgrin.ai;

import com.studio.dread.entity.pitchgrin.PitchGrinEntity;
import com.studio.dread.registry.ModSounds;
import net.minecraft.entity.ai.goal.Goal;
import net.minecraft.entity.player.PlayerEntity;
import net.minecraft.util.math.BlockPos;
import net.minecraft.world.World;

import java.util.EnumSet;

public class CeilingLurkGoal extends Goal {
    private final PitchGrinEntity pitchGrin;
    private BlockPos ceilingAnchor;
    private int lurkTicks = 0;

    public CeilingLurkGoal(PitchGrinEntity pitchGrin) {
        this.pitchGrin = pitchGrin;
        this.setControls(EnumSet.of(Control.MOVE, Control.JUMP));
    }

    @Override
    public boolean canStart() {
        if (pitchGrin.isCeilingClinging()) {
            return true;
        }
        PlayerEntity nearest = pitchGrin.getWorld().getClosestPlayer(pitchGrin, 28.0);
        if (nearest == null || pitchGrin.squaredDistanceTo(nearest) < 8.0 * 8.0) {
            return false;
        }
        this.ceilingAnchor = findCeilingAbove(pitchGrin.getBlockPos());
        return this.ceilingAnchor != null;
    }

    @Override
    public void start() {
        this.lurkTicks = 0;
        if (ceilingAnchor != null) {
            pitchGrin.refreshPositionAfterTeleport(
                    ceilingAnchor.getX() + 0.5,
                    ceilingAnchor.getY() - 1.5,
                    ceilingAnchor.getZ() + 0.5
            );
            pitchGrin.setCeilingClinging(true);
        }
    }

    @Override
    public boolean shouldContinue() {
        if (!pitchGrin.isCeilingClinging() || lurkTicks > 300) {
            return false;
        }
        PlayerEntity nearest = pitchGrin.getWorld().getClosestPlayer(pitchGrin, 24.0);
        return nearest != null && nearest.isAlive();
    }

    @Override
    public void tick() {
        lurkTicks++;
        PlayerEntity nearest = pitchGrin.getWorld().getClosestPlayer(pitchGrin, 24.0);
        if (nearest == null) return;

        pitchGrin.getLookControl().lookAt(nearest, 30.0f, 30.0f);

        // Oyuncu tavan pususunun altından geçerken (yatay mesafe < 4.5 blok) üzerine atlar
        double dx = nearest.getX() - pitchGrin.getX();
        double dz = nearest.getZ() - pitchGrin.getZ();
        if (dx * dx + dz * dz <= 4.5 * 4.5) {
            pitchGrin.setCeilingClinging(false);
            pitchGrin.playSound(ModSounds.PITCH_GRIN_SHRIEK_BITE, 1.4f, 0.8f);
            pitchGrin.setVelocity(dx * 0.25, -0.6, dz * 0.25);
        }
    }

    @Override
    public void stop() {
        pitchGrin.setCeilingClinging(false);
    }

    private BlockPos findCeilingAbove(BlockPos start) {
        World world = pitchGrin.getWorld();
        for (int dy = 3; dy <= 10; dy++) {
            BlockPos check = start.up(dy);
            if (world.getBlockState(check).isSolidBlock(world, check) && world.getBlockState(check.down()).isAir()) {
                return check;
            }
        }
        return null;
    }
}
```

## Dosya: `src/main/java/com/studio/dread/entity/pitchgrin/ai/TunnelHuntGoal.java`
```java
package com.studio.dread.entity.pitchgrin.ai;

import com.studio.dread.entity.pitchgrin.PitchGrinEntity;
import net.minecraft.entity.ai.goal.Goal;
import net.minecraft.entity.player.PlayerEntity;
import net.minecraft.util.Hand;
import net.minecraft.util.math.Vec3d;

import java.util.EnumSet;

public class TunnelHuntGoal extends Goal {
    private final PitchGrinEntity pitchGrin;
    private PlayerEntity target;

    public TunnelHuntGoal(PitchGrinEntity pitchGrin) {
        this.pitchGrin = pitchGrin;
        this.setControls(EnumSet.of(Control.MOVE, Control.LOOK));
    }

    @Override
    public boolean canStart() {
        if (pitchGrin.isCeilingClinging()) return false;
        this.target = pitchGrin.getWorld().getClosestPlayer(pitchGrin, 42.0);
        return this.target != null && this.target.isAlive() && !this.target.isCreative();
    }

    @Override
    public boolean shouldContinue() {
        return !pitchGrin.isCeilingClinging() && target != null && target.isAlive() && !target.isCreative();
    }

    @Override
    public void tick() {
        if (target == null) return;
        pitchGrin.getLookControl().lookAt(target, 30.0f, 30.0f);

        // Oyuncu doğrudan Pitch Grin'e bakıyorsa ve mesafe > 8 bloksa anlık donarak gerilim yaratır
        Vec3d playerLook = target.getRotationVec(1.0f).normalize();
        Vec3d toEntity = pitchGrin.getEyePos().subtract(target.getEyePos()).normalize();
        double distSq = pitchGrin.squaredDistanceTo(target);

        if (playerLook.dotProduct(toEntity) > 0.90 && distSq > 8.0 * 8.0 && distSq < 22.0 * 22.0) {
            pitchGrin.getNavigation().stop();
            pitchGrin.setSprinting(false);
            return;
        }

        // Tünel avı: Dört uzuv üzerinde hızla sürünerek yaklaşır
        pitchGrin.setCrawlingMode(true);
        pitchGrin.setSprinting(distSq < 14.0 * 14.0);
        pitchGrin.getNavigation().startMovingTo(target, pitchGrin.isSprinting() ? 1.55 : 1.20);

        if (distSq <= 2.8 * 2.8 && pitchGrin.age % 15 == 0) {
            pitchGrin.swingHand(Hand.MAIN_HAND);
            pitchGrin.tryAttack(target);
        }
    }
}
```

## Dosya: `src/main/java/com/studio/dread/entity/stalker/GrinningStalkerEntity.java`
```java
package com.studio.dread.entity.stalker;

import com.studio.dread.entity.stalker.ai.BaseWreckerGoal;
import com.studio.dread.entity.stalker.ai.BrinkOfDeathGoal;
import com.studio.dread.entity.stalker.ai.ExecutionerGoal;
import com.studio.dread.entity.stalker.ai.HitAndRunGoal;
import com.studio.dread.entity.stalker.ai.WatcherGoal;
import com.studio.dread.registry.ModItems;
import com.studio.dread.registry.ModSounds;
import net.minecraft.block.BlockState;
import net.minecraft.entity.Entity;
import net.minecraft.entity.EntityDimensions;
import net.minecraft.entity.EntityPose;
import net.minecraft.entity.EntityType;
import net.minecraft.entity.ai.goal.LookAroundGoal;
import net.minecraft.entity.ai.goal.LookAtEntityGoal;
import net.minecraft.entity.ai.goal.SwimGoal;
import net.minecraft.entity.attribute.DefaultAttributeContainer;
import net.minecraft.entity.attribute.EntityAttributes;
import net.minecraft.entity.damage.DamageSource;
import net.minecraft.entity.data.DataTracker;
import net.minecraft.entity.data.TrackedData;
import net.minecraft.entity.data.TrackedDataHandlerRegistry;
import net.minecraft.entity.mob.HostileEntity;
import net.minecraft.entity.player.PlayerEntity;
import net.minecraft.entity.vehicle.BoatEntity;
import net.minecraft.entity.vehicle.MinecartEntity;
import net.minecraft.nbt.NbtCompound;
import net.minecraft.particle.ParticleTypes;
import net.minecraft.server.world.ServerWorld;
import net.minecraft.sound.SoundEvent;
import net.minecraft.util.math.BlockPos;
import net.minecraft.util.math.Vec3d;
import net.minecraft.world.World;
import software.bernie.geckolib.animatable.GeoEntity;
import software.bernie.geckolib.animatable.instance.AnimatableInstanceCache;
import software.bernie.geckolib.animation.AnimatableManager;
import software.bernie.geckolib.animation.AnimationController;
import software.bernie.geckolib.animation.AnimationState;
import software.bernie.geckolib.animation.PlayState;
import software.bernie.geckolib.animation.RawAnimation;
import software.bernie.geckolib.util.GeckoLibUtil;

public class GrinningStalkerEntity extends HostileEntity implements GeoEntity {
    private static final TrackedData<Integer> VARIANT_ID = DataTracker.registerData(GrinningStalkerEntity.class, TrackedDataHandlerRegistry.INTEGER);
    private static final TrackedData<Integer> STALKER_POSE_ID = DataTracker.registerData(GrinningStalkerEntity.class, TrackedDataHandlerRegistry.INTEGER);
    private static final TrackedData<Boolean> CLIMBING_WALL = DataTracker.registerData(GrinningStalkerEntity.class, TrackedDataHandlerRegistry.BOOLEAN);

    // 3.5 blok boy, 1.4/1.5 blok genişlik (Dik yürüyüş) vs 1.5 blok boy, 1.3 blok genişlik (2 blokluk geçitlerden sürünerek geçme)
    private static final EntityDimensions UPRIGHT_DIMENSIONS = EntityDimensions.changing(1.4f, 3.5f).withEyeHeight(3.15f);
    private static final EntityDimensions CRAWLING_DIMENSIONS = EntityDimensions.changing(1.3f, 1.5f).withEyeHeight(1.15f);

    private static final RawAnimation IDLE_ANIM = RawAnimation.begin().thenLoop("animation.dread.idle");
    private static final RawAnimation WATCHING_ANIM = RawAnimation.begin().thenLoop("animation.dread.watching");
    private static final RawAnimation WALK_ANIM = RawAnimation.begin().thenLoop("animation.dread.walk");
    private static final RawAnimation SPRINT_CRAWL_ANIM = RawAnimation.begin().thenLoop("animation.dread.sprint");
    private static final RawAnimation CRAWL_ANIM = RawAnimation.begin().thenLoop("animation.dread.crawl");
    private static final RawAnimation CLIMB_ANIM = RawAnimation.begin().thenLoop("animation.dread.climb");
    private static final RawAnimation OBSERVE_ANIM = RawAnimation.begin().thenPlayAndHold("animation.dread.observe");
    private static final RawAnimation ATTACK_ANIM = RawAnimation.begin().thenPlay("animation.dread.attack");
    private static final RawAnimation VANISH_ANIM = RawAnimation.begin().thenPlay("animation.dread.vanish");

    private final AnimatableInstanceCache geoCache = GeckoLibUtil.createInstanceCache(this);
    private int vanishTicks = 0;

    public GrinningStalkerEntity(EntityType<? extends HostileEntity> entityType, World world) {
        super(entityType, world);
        this.experiencePoints = 25;
    }

    public static DefaultAttributeContainer.Builder createStalkerAttributes() {
        return HostileEntity.createHostileAttributes()
                .add(EntityAttributes.GENERIC_MAX_HEALTH, 120.0)
                .add(EntityAttributes.GENERIC_MOVEMENT_SPEED, 0.28)
                .add(EntityAttributes.GENERIC_ATTACK_DAMAGE, 8.0)
                .add(EntityAttributes.GENERIC_FOLLOW_RANGE, 64.0)
                .add(EntityAttributes.GENERIC_KNOCKBACK_RESISTANCE, 0.75)
                .add(EntityAttributes.GENERIC_STEP_HEIGHT, 1.25);
    }

    @Override
    protected void initDataTracker(DataTracker.Builder builder) {
        super.initDataTracker(builder);
        builder.add(VARIANT_ID, StalkerVariant.WATCHER.getId());
        builder.add(STALKER_POSE_ID, StalkerPose.UPRIGHT.getId());
        builder.add(CLIMBING_WALL, false);
    }

    @Override
    protected void initGoals() {
        this.goalSelector.add(0, new SwimGoal(this));
        this.goalSelector.add(1, new WatcherGoal(this));
        this.goalSelector.add(1, new HitAndRunGoal(this));
        this.goalSelector.add(1, new BaseWreckerGoal(this));
        this.goalSelector.add(1, new BrinkOfDeathGoal(this));
        this.goalSelector.add(1, new ExecutionerGoal(this));
        this.goalSelector.add(8, new LookAtEntityGoal(this, PlayerEntity.class, 48.0f));
        this.goalSelector.add(9, new LookAroundGoal(this));
    }

    public StalkerVariant getVariant() {
        return StalkerVariant.byId(this.dataTracker.get(VARIANT_ID));
    }

    public void setVariant(StalkerVariant variant) {
        this.dataTracker.set(VARIANT_ID, variant.getId());
        if (this.getAttributeInstance(EntityAttributes.GENERIC_MAX_HEALTH) != null) {
            this.getAttributeInstance(EntityAttributes.GENERIC_MAX_HEALTH).setBaseValue(variant.getMaxHealth());
            this.setHealth(variant.getMaxHealth());
        }
        if (this.getAttributeInstance(EntityAttributes.GENERIC_ATTACK_DAMAGE) != null) {
            this.getAttributeInstance(EntityAttributes.GENERIC_ATTACK_DAMAGE).setBaseValue(variant.getAttackDamage());
        }
        if (this.getAttributeInstance(EntityAttributes.GENERIC_MOVEMENT_SPEED) != null) {
            this.getAttributeInstance(EntityAttributes.GENERIC_MOVEMENT_SPEED).setBaseValue(variant.getBaseSpeed());
        }
    }

    public StalkerPose getStalkerPose() {
        return StalkerPose.byId(this.dataTracker.get(STALKER_POSE_ID));
    }

    public void setStalkerPose(StalkerPose pose) {
        StalkerPose current = getStalkerPose();
        if (current != pose) {
            this.dataTracker.set(STALKER_POSE_ID, pose.getId());
            this.calculateDimensions();
        }
    }

    @Override
    public void onTrackedDataSet(TrackedData<?> data) {
        super.onTrackedDataSet(data);
        if (STALKER_POSE_ID.equals(data)) {
            this.calculateDimensions();
        }
    }

    @Override
    protected EntityDimensions getBaseDimensions(EntityPose pose) {
        StalkerPose stalkerPose = getStalkerPose();
        if (stalkerPose == StalkerPose.CRAWLING || stalkerPose == StalkerPose.OBSERVING) {
            return CRAWLING_DIMENSIONS;
        }
        return UPRIGHT_DIMENSIONS;
    }

    @Override
    public void tick() {
        super.tick();

        if (!this.getWorld().isClient()) {
            // Vanish animasyonu tamamlandığında sil
            if (getStalkerPose() == StalkerPose.VANISHING) {
                vanishTicks++;
                if (vanishTicks >= 10) {
                    if (this.getWorld() instanceof ServerWorld serverWorld) {
                        serverWorld.spawnParticles(ParticleTypes.LARGE_SMOKE, getX(), getY() + 1.2, getZ(), 35, 0.5, 0.9, 0.5, 0.04);
                        serverWorld.spawnParticles(ParticleTypes.ASH, getX(), getY() + 1.2, getZ(), 50, 0.6, 1.0, 0.6, 0.02);
                    }
                    this.discard();
                    return;
                }
            }

            // Anti-Cheese: Tekne veya Minecart temasını anında kır
            for (Entity nearby : this.getWorld().getOtherEntities(this, this.getBoundingBox().expand(0.6))) {
                if (nearby instanceof BoatEntity || nearby instanceof MinecartEntity) {
                    nearby.kill();
                }
            }

            // Duvar tırmanma kontrolü (Oyuncu yukarı kule yaptığında)
            this.dataTracker.set(CLIMBING_WALL, this.horizontalCollision);

            // Dinamik 2 blokluk geçit ve dört uzuv koşu kontrolü (Her 5 tick'te bir)
            if (this.age % 5 == 0 && getStalkerPose() != StalkerPose.VANISHING && getStalkerPose() != StalkerPose.OBSERVING) {
                updateDynamicPosture();
            }
        }
    }

    private void updateDynamicPosture() {
        if (this.dataTracker.get(CLIMBING_WALL)) {
            setStalkerPose(StalkerPose.CLIMBING);
            return;
        }

        boolean needsLowClearance = hasLowCeilingAheadOrAbove();
        boolean isSprintingFast = this.isSprinting() || this.getVelocity().horizontalLengthSquared() > 0.035;

        if (needsLowClearance || isSprintingFast) {
            setStalkerPose(StalkerPose.CRAWLING);
        } else {
            setStalkerPose(StalkerPose.UPRIGHT);
        }
    }

    private boolean hasLowCeilingAheadOrAbove() {
        World world = this.getWorld();
        BlockPos base = this.getBlockPos();
        // Kendi üstündeki 3. ve 4. blokları kontrol et
        if (!world.getBlockState(base.up(2)).isAir() || !world.getBlockState(base.up(3)).isAir()) {
            return true;
        }
        // İlerlediği yöndeki 2 blokluk geçidi kontrol et
        Vec3d forward = this.getRotationVec(1.0f).normalize();
        BlockPos front = BlockPos.ofFloored(this.getX() + forward.x * 1.2, this.getY(), this.getZ() + forward.z * 1.2);
        boolean bottomOpen = world.getBlockState(front).isAir() && world.getBlockState(front.up(1)).isAir();
        boolean topBlocked = !world.getBlockState(front.up(2)).isAir() || !world.getBlockState(front.up(3)).isAir();
        return bottomOpen && topBlocked;
    }

    public void triggerVanish() {
        if (getStalkerPose() != StalkerPose.VANISHING) {
            setStalkerPose(StalkerPose.VANISHING);
            this.playSound(ModSounds.STALKER_VANISH, 1.2f, 0.85f);
        }
    }

    @Override
    public boolean isClimbing() {
        return this.dataTracker.get(CLIMBING_WALL);
    }

    @Override
    protected boolean canStartRiding(Entity entity) {
        return false; // Tekne/vagon tuzağına girmez
    }

    @Override
    public boolean isFireImmune() {
        return true;
    }

    @Override
    protected void dropLoot(DamageSource damageSource, boolean causedByPlayer) {
        super.dropLoot(damageSource, causedByPlayer);
        if (causedByPlayer && (getVariant() == StalkerVariant.BRINK_OF_DEATH || getVariant() == StalkerVariant.EXECUTIONER)) {
            this.dropItem(ModItems.GRINNING_HEART);
        }
    }

    @Override
    protected SoundEvent getAmbientSound() {
        return ModSounds.STALKER_BREATHING;
    }

    @Override
    protected SoundEvent getHurtSound(DamageSource source) {
        return ModSounds.STALKER_BONE_CLICK;
    }

    @Override
    protected SoundEvent getDeathSound() {
        return ModSounds.STALKER_VANISH;
    }

    @Override
    protected void playStepSound(BlockPos pos, BlockState state) {
        this.playSound(ModSounds.STALKER_FOOTSTEP_HEAVY, 0.45f, 0.9f + this.random.nextFloat() * 0.2f);
    }

    @Override
    public void writeCustomDataToNbt(NbtCompound nbt) {
        super.writeCustomDataToNbt(nbt);
        nbt.putInt("StalkerVariant", getVariant().getId());
        nbt.putInt("StalkerPose", getStalkerPose().getId());
    }

    @Override
    public void readCustomDataFromNbt(NbtCompound nbt) {
        super.readCustomDataFromNbt(nbt);
        if (nbt.contains("StalkerVariant")) {
            setVariant(StalkerVariant.byId(nbt.getInt("StalkerVariant")));
        }
        if (nbt.contains("StalkerPose")) {
            setStalkerPose(StalkerPose.byId(nbt.getInt("StalkerPose")));
        }
    }

    @Override
    public void registerControllers(AnimatableManager.ControllerRegistrar controllers) {
        controllers.add(new AnimationController<>(this, "main_controller", 3, this::predicate));
    }

    private PlayState predicate(AnimationState<GrinningStalkerEntity> state) {
        StalkerPose pose = getStalkerPose();
        if (pose == StalkerPose.VANISHING) {
            return state.setAndContinue(VANISH_ANIM);
        }
        if (pose == StalkerPose.OBSERVING) {
            return state.setAndContinue(OBSERVE_ANIM);
        }
        if (pose == StalkerPose.CLIMBING) {
            return state.setAndContinue(CLIMB_ANIM);
        }
        if (this.handSwinging) {
            return state.setAndContinue(ATTACK_ANIM);
        }
        if (state.isMoving()) {
            if (pose == StalkerPose.CRAWLING) {
                return state.setAndContinue(this.isSprinting() ? SPRINT_CRAWL_ANIM : CRAWL_ANIM);
            }
            return state.setAndContinue(WALK_ANIM);
        }
        if (getVariant() == StalkerVariant.WATCHER) {
            return state.setAndContinue(WATCHING_ANIM);
        }
        return state.setAndContinue(IDLE_ANIM);
    }

    @Override
    public AnimatableInstanceCache getAnimatableInstanceCache() {
        return geoCache;
    }
}
```

## Dosya: `src/main/java/com/studio/dread/entity/stalker/StalkerPose.java`
```java
package com.studio.dread.entity.stalker;

public enum StalkerPose {
    UPRIGHT(0),
    CRAWLING(1),
    CLIMBING(2),
    OBSERVING(3),
    VANISHING(4);

    private final int id;

    StalkerPose(int id) {
        this.id = id;
    }

    public int getId() {
        return id;
    }

    public static StalkerPose byId(int id) {
        for (StalkerPose p : values()) {
            if (p.id == id) return p;
        }
        return UPRIGHT;
    }
}
```

## Dosya: `src/main/java/com/studio/dread/entity/stalker/StalkerVariant.java`
```java
package com.studio.dread.entity.stalker;

public enum StalkerVariant {
    WATCHER(0, "watcher", 60.0f, 0.0f, 0.24f),
    HIT_AND_RUN(1, "hit_and_run", 80.0f, 6.0f, 0.28f),
    BASE_WRECKER(2, "base_wrecker", 120.0f, 8.0f, 0.26f),
    BRINK_OF_DEATH(3, "brink_of_death", 160.0f, 10.0f, 0.30f),
    EXECUTIONER(4, "executioner", 220.0f, 14.0f, 0.32f);

    private final int id;
    private final String name;
    private final float maxHealth;
    private final float attackDamage;
    private final float baseSpeed;

    StalkerVariant(int id, String name, float maxHealth, float attackDamage, float baseSpeed) {
        this.id = id;
        this.name = name;
        this.maxHealth = maxHealth;
        this.attackDamage = attackDamage;
        this.baseSpeed = baseSpeed;
    }

    public int getId() {
        return id;
    }

    public String getName() {
        return name;
    }

    public float getMaxHealth() {
        return maxHealth;
    }

    public float getAttackDamage() {
        return attackDamage;
    }

    public float getBaseSpeed() {
        return baseSpeed;
    }

    public static StalkerVariant byId(int id) {
        for (StalkerVariant v : values()) {
            if (v.id == id) return v;
        }
        return WATCHER;
    }
}
```

## Dosya: `src/main/java/com/studio/dread/entity/stalker/ai/BaseWreckerGoal.java`
```java
package com.studio.dread.entity.stalker.ai;

import com.studio.dread.config.DreadConfig;
import com.studio.dread.core.dread.DreadManager;
import com.studio.dread.entity.stalker.GrinningStalkerEntity;
import com.studio.dread.entity.stalker.StalkerVariant;
import com.studio.dread.registry.ModSounds;
import net.minecraft.block.Block;
import net.minecraft.block.BlockState;
import net.minecraft.block.Blocks;
import net.minecraft.block.CampfireBlock;
import net.minecraft.block.DoorBlock;
import net.minecraft.block.StainedGlassBlock;
import net.minecraft.block.StainedGlassPaneBlock;
import net.minecraft.block.TrapdoorBlock;
import net.minecraft.entity.ai.goal.Goal;
import net.minecraft.entity.player.PlayerEntity;
import net.minecraft.server.network.ServerPlayerEntity;
import net.minecraft.sound.SoundCategory;
import net.minecraft.state.property.Properties;
import net.minecraft.util.Hand;
import net.minecraft.util.math.BlockPos;

import java.util.EnumSet;

public class BaseWreckerGoal extends Goal {
    private final GrinningStalkerEntity stalker;
    private BlockPos currentTargetBlock;
    private int scanCooldown = 0;
    private int activeTicks = 0;
    private static final int MAX_SIEGE_TICKS = 1800; // 90 saniye

    public BaseWreckerGoal(GrinningStalkerEntity stalker) {
        this.stalker = stalker;
        this.setControls(EnumSet.of(Control.MOVE, Control.LOOK));
    }

    @Override
    public boolean canStart() {
        return stalker.getVariant() == StalkerVariant.BASE_WRECKER;
    }

    @Override
    public boolean shouldContinue() {
        return stalker.getVariant() == StalkerVariant.BASE_WRECKER && activeTicks < MAX_SIEGE_TICKS;
    }

    @Override
    public void tick() {
        activeTicks++;
        if (stalker.getWorld().isDay() && activeTicks > 200) {
            stalker.triggerVanish();
            return;
        }

        if (--scanCooldown <= 0 || currentTargetBlock == null || !isValidTarget(stalker.getWorld().getBlockState(currentTargetBlock))) {
            scanCooldown = 30;
            currentTargetBlock = findHighestPriorityTarget();
        }

        if (currentTargetBlock != null) {
            stalker.getLookControl().lookAt(
                    currentTargetBlock.getX() + 0.5,
                    currentTargetBlock.getY() + 0.5,
                    currentTargetBlock.getZ() + 0.5
            );
            stalker.getNavigation().startMovingTo(
                    currentTargetBlock.getX() + 0.5,
                    currentTargetBlock.getY(),
                    currentTargetBlock.getZ() + 0.5,
                    1.35
            );

            if (stalker.getBlockPos().getSquaredDistance(currentTargetBlock) <= 9.0) {
                sabotageBlock(currentTargetBlock);
                currentTargetBlock = null;
                scanCooldown = 10;
            }
        } else {
            // Işık kalmadıysa en yakın oyuncuyu sıkıştır
            PlayerEntity nearest = stalker.getWorld().getClosestPlayer(stalker, 32.0);
            if (nearest != null) {
                stalker.getNavigation().startMovingTo(nearest, 1.35);
                if (stalker.squaredDistanceTo(nearest) <= 3.0 * 3.0) {
                    stalker.swingHand(Hand.MAIN_HAND);
                    stalker.tryAttack(nearest);
                }
            } else if (activeTicks > 400) {
                stalker.triggerVanish();
            }
        }
    }

    /**
     * Hedef Önceliği:
     * 1. Işık Kaynakları (Torch, Soul Torch, Lantern, Campfire, Glowstone)
     * 2. Kapılar (Açık değilse zorla açılır)
     * 3. Camlar (Glass, Glass Pane)
     */
    private BlockPos findHighestPriorityTarget() {
        BlockPos origin = stalker.getBlockPos();
        BlockPos bestLight = null;
        double bestLightDist = Double.MAX_VALUE;
        BlockPos bestDoorOrGlass = null;
        double bestSecondaryDist = Double.MAX_VALUE;

        for (BlockPos pos : BlockPos.iterateOutwards(origin, 24, 8, 24)) {
            BlockState state = stalker.getWorld().getBlockState(pos);
            if (state.isAir()) continue;

            double dist = origin.getSquaredDistance(pos);
            if (isBreakableLightSource(state)) {
                if (dist < bestLightDist) {
                    bestLightDist = dist;
                    bestLight = pos.toImmutable();
                }
            } else if (bestLight == null && (isClosedDoor(state) || isGlassBlock(state))) {
                if (dist < bestSecondaryDist) {
                    bestSecondaryDist = dist;
                    bestDoorOrGlass = pos.toImmutable();
                }
            }
        }

        return bestLight != null ? bestLight : bestDoorOrGlass;
    }

    private boolean isValidTarget(BlockState state) {
        return isBreakableLightSource(state) || isClosedDoor(state) || isGlassBlock(state);
    }

    public static boolean isBreakableLightSource(BlockState state) {
        Block b = state.getBlock();
        if (b == Blocks.TORCH || b == Blocks.WALL_TORCH
                || b == Blocks.SOUL_TORCH || b == Blocks.SOUL_WALL_TORCH
                || b == Blocks.LANTERN || b == Blocks.SOUL_LANTERN
                || b == Blocks.GLOWSTONE) {
            return true;
        }
        if (b instanceof CampfireBlock && state.contains(Properties.LIT)) {
            return state.get(Properties.LIT);
        }
        return false;
    }

    private boolean isClosedDoor(BlockState state) {
        if (state.getBlock() instanceof DoorBlock && state.contains(Properties.OPEN)) {
            return !state.get(Properties.OPEN);
        }
        if (state.getBlock() instanceof TrapdoorBlock && state.contains(Properties.OPEN)) {
            return !state.get(Properties.OPEN);
        }
        return false;
    }

    private boolean isGlassBlock(BlockState state) {
        Block b = state.getBlock();
        return b == Blocks.GLASS || b == Blocks.GLASS_PANE
                || b == Blocks.TINTED_GLASS
                || b instanceof StainedGlassBlock
                || b instanceof StainedGlassPaneBlock;
    }

    private void sabotageBlock(BlockPos pos) {
        BlockState state = stalker.getWorld().getBlockState(pos);
        stalker.swingHand(Hand.MAIN_HAND);

        if (isBreakableLightSource(state)) {
            stalker.getWorld().breakBlock(pos, true, stalker);
            stalker.getWorld().playSound(null, pos, ModSounds.AMBIENT_FAKE_TORCH_HISS, SoundCategory.HOSTILE, 1.1f, 0.7f);
            // Yakındaki oyunculara +10 Dread ekle
            for (PlayerEntity p : stalker.getWorld().getPlayers()) {
                if (p instanceof ServerPlayerEntity sp && sp.squaredDistanceTo(pos.toCenterPos()) <= 36.0 * 36.0) {
                    DreadManager.addEventDread(sp, DreadConfig.TORCH_BREAK_DREAD_BONUS);
                }
            }
        } else if (isClosedDoor(state)) {
            stalker.getWorld().setBlockState(pos, state.with(Properties.OPEN, true), Block.NOTIFY_ALL);
            stalker.getWorld().playSound(null, pos, ModSounds.STALKER_SLASH, SoundCategory.HOSTILE, 1.0f, 0.8f);
        } else if (isGlassBlock(state)) {
            stalker.getWorld().breakBlock(pos, false, stalker);
        }
    }
}
```

## Dosya: `src/main/java/com/studio/dread/entity/stalker/ai/BrinkOfDeathGoal.java`
```java
package com.studio.dread.entity.stalker.ai;

import com.studio.dread.entity.stalker.GrinningStalkerEntity;
import com.studio.dread.entity.stalker.StalkerPose;
import com.studio.dread.entity.stalker.StalkerVariant;
import com.studio.dread.registry.ModAdvancementCriteria;
import com.studio.dread.registry.ModSounds;
import net.minecraft.entity.ai.goal.Goal;
import net.minecraft.entity.player.PlayerEntity;
import net.minecraft.server.network.ServerPlayerEntity;
import net.minecraft.util.Hand;

import java.util.EnumSet;

public class BrinkOfDeathGoal extends Goal {
    private final GrinningStalkerEntity stalker;
    private PlayerEntity target;
    private int observeCountdown = 0;

    public BrinkOfDeathGoal(GrinningStalkerEntity stalker) {
        this.stalker = stalker;
        this.setControls(EnumSet.of(Control.MOVE, Control.LOOK));
    }

    @Override
    public boolean canStart() {
        if (stalker.getVariant() != StalkerVariant.BRINK_OF_DEATH) {
            return false;
        }
        this.target = stalker.getWorld().getClosestPlayer(stalker, 56.0);
        return this.target != null && this.target.isAlive() && !this.target.isCreative();
    }

    @Override
    public boolean shouldContinue() {
        return stalker.getVariant() == StalkerVariant.BRINK_OF_DEATH
                && target != null
                && target.isAlive();
    }

    @Override
    public void tick() {
        if (target == null) return;
        stalker.getLookControl().lookAt(target, 30.0f, 30.0f);

        // Oyuncu yarım kalbe (1.0 HP) düştüyse son darbeyi vurmaz; observe animasyonuyla izleyip kaybolur
        if (target.getHealth() <= 1.0f || observeCountdown > 0) {
            stalker.getNavigation().stop();
            stalker.setStalkerPose(StalkerPose.OBSERVING);
            if (observeCountdown == 0) {
                observeCountdown = 45; // 2.25 saniye oyuncunun yüzüne eğilip sırıtır
                stalker.playSound(ModSounds.STALKER_BONE_CLICK, 1.2f, 0.6f);
                if (target instanceof ServerPlayerEntity serverPlayer) {
                    ModAdvancementCriteria.SURVIVE_HALF_HEART.trigger(serverPlayer);
                }
            } else if (--observeCountdown <= 1) {
                stalker.triggerVanish();
            }
            return;
        }

        stalker.setSprinting(true);
        stalker.getNavigation().startMovingTo(target, 1.55);

        if (stalker.squaredDistanceTo(target) <= 3.2 * 3.2 && stalker.age % 15 == 0) {
            stalker.swingHand(Hand.MAIN_HAND);
            stalker.tryAttack(target);
        }
    }
}
```

## Dosya: `src/main/java/com/studio/dread/entity/stalker/ai/ExecutionerGoal.java`
```java
package com.studio.dread.entity.stalker.ai;

import com.studio.dread.entity.stalker.GrinningStalkerEntity;
import com.studio.dread.entity.stalker.StalkerVariant;
import com.studio.dread.registry.ModAdvancementCriteria;
import net.minecraft.entity.ai.goal.Goal;
import net.minecraft.entity.effect.StatusEffectInstance;
import net.minecraft.entity.effect.StatusEffects;
import net.minecraft.entity.player.PlayerEntity;
import net.minecraft.server.network.ServerPlayerEntity;
import net.minecraft.util.Hand;

import java.util.EnumSet;

public class ExecutionerGoal extends Goal {
    private final GrinningStalkerEntity stalker;
    private PlayerEntity target;

    public ExecutionerGoal(GrinningStalkerEntity stalker) {
        this.stalker = stalker;
        this.setControls(EnumSet.of(Control.MOVE, Control.LOOK));
    }

    @Override
    public boolean canStart() {
        if (stalker.getVariant() != StalkerVariant.EXECUTIONER) {
            return false;
        }
        this.target = stalker.getWorld().getClosestPlayer(stalker, 64.0);
        return this.target != null && this.target.isAlive() && !this.target.isCreative();
    }

    @Override
    public boolean shouldContinue() {
        return stalker.getVariant() == StalkerVariant.EXECUTIONER
                && target != null
                && target.isAlive();
    }

    @Override
    public void tick() {
        if (target == null) return;
        stalker.getLookControl().lookAt(target, 30.0f, 30.0f);

        boolean isHardcore = stalker.getWorld().getLevelProperties().isHardcore();

        // Hardcore ise ve oyuncu yarım kalbe (1.0 HP) indiyse Darkness verip kaybolur
        if (isHardcore && target.getHealth() <= 1.0f) {
            target.addStatusEffect(new StatusEffectInstance(StatusEffects.DARKNESS, 240, 0));
            target.addStatusEffect(new StatusEffectInstance(StatusEffects.BLINDNESS, 60, 0));
            if (target instanceof ServerPlayerEntity serverPlayer) {
                ModAdvancementCriteria.SURVIVE_HALF_HEART.trigger(serverPlayer);
            }
            stalker.triggerVanish();
            return;
        }

        stalker.setSprinting(true);
        stalker.getNavigation().startMovingTo(target, 1.70);

        if (stalker.squaredDistanceTo(target) <= 3.3 * 3.3 && stalker.age % 14 == 0) {
            stalker.swingHand(Hand.MAIN_HAND);
            stalker.tryAttack(target);
        }
    }
}
```

## Dosya: `src/main/java/com/studio/dread/entity/stalker/ai/HitAndRunGoal.java`
```java
package com.studio.dread.entity.stalker.ai;

import com.studio.dread.config.DreadConfig;
import com.studio.dread.core.dread.DreadManager;
import com.studio.dread.entity.stalker.GrinningStalkerEntity;
import com.studio.dread.entity.stalker.StalkerPose;
import com.studio.dread.entity.stalker.StalkerVariant;
import com.studio.dread.registry.ModSounds;
import net.minecraft.entity.ai.goal.Goal;
import net.minecraft.entity.effect.StatusEffectInstance;
import net.minecraft.entity.effect.StatusEffects;
import net.minecraft.entity.player.PlayerEntity;
import net.minecraft.server.network.ServerPlayerEntity;
import net.minecraft.util.Hand;

import java.util.EnumSet;

public class HitAndRunGoal extends Goal {
    private final GrinningStalkerEntity stalker;
    private PlayerEntity target;
    private int chaseTicks = 0;
    private boolean hasStruck = false;

    public HitAndRunGoal(GrinningStalkerEntity stalker) {
        this.stalker = stalker;
        this.setControls(EnumSet.of(Control.MOVE, Control.LOOK));
    }

    @Override
    public boolean canStart() {
        if (stalker.getVariant() != StalkerVariant.HIT_AND_RUN) {
            return false;
        }
        this.target = stalker.getWorld().getClosestPlayer(stalker, 48.0);
        return this.target != null && this.target.isAlive() && !this.target.isCreative();
    }

    @Override
    public void start() {
        this.chaseTicks = 0;
        this.hasStruck = false;
        this.stalker.setSprinting(true);
        this.stalker.setStalkerPose(StalkerPose.CRAWLING);
    }

    @Override
    public boolean shouldContinue() {
        return stalker.getVariant() == StalkerVariant.HIT_AND_RUN
                && !hasStruck
                && target != null
                && target.isAlive()
                && chaseTicks < 200; // Maksimum 10 saniye takip
    }

    @Override
    public void tick() {
        chaseTicks++;
        if (target == null) return;

        stalker.getLookControl().lookAt(target, 30.0f, 30.0f);
        stalker.getNavigation().startMovingTo(target, 1.65);

        double distSq = stalker.squaredDistanceTo(target);
        if (distSq <= 3.2 * 3.2) {
            stalker.swingHand(Hand.MAIN_HAND);
            stalker.tryAttack(target);

            // Darkness verir ve Dread artırır
            target.addStatusEffect(new StatusEffectInstance(StatusEffects.DARKNESS, 160, 0));
            target.addStatusEffect(new StatusEffectInstance(StatusEffects.SLOWNESS, 50, 1));
            if (target instanceof ServerPlayerEntity serverPlayer) {
                DreadManager.addEventDread(serverPlayer, DreadConfig.HIT_AND_RUN_DREAD_BONUS);
            }

            stalker.playSound(ModSounds.STALKER_SLASH, 1.2f, 0.9f);
            hasStruck = true;
            stalker.triggerVanish();
        } else if (chaseTicks >= 195) {
            stalker.triggerVanish();
        }
    }
}
```

## Dosya: `src/main/java/com/studio/dread/entity/stalker/ai/WatcherGoal.java`
```java
package com.studio.dread.entity.stalker.ai;

import com.studio.dread.config.DreadConfig;
import com.studio.dread.core.dread.DreadManager;
import com.studio.dread.entity.stalker.GrinningStalkerEntity;
import com.studio.dread.entity.stalker.StalkerVariant;
import com.studio.dread.network.HallucinationEventS2CPayload;
import com.studio.dread.registry.ModSounds;
import net.fabricmc.fabric.api.networking.v1.ServerPlayNetworking;
import net.minecraft.block.Block;
import net.minecraft.block.BlockState;
import net.minecraft.block.Blocks;
import net.minecraft.entity.ai.goal.Goal;
import net.minecraft.entity.player.PlayerEntity;
import net.minecraft.server.network.ServerPlayerEntity;
import net.minecraft.sound.SoundCategory;
import net.minecraft.util.math.BlockPos;
import net.minecraft.util.math.Vec3d;

import java.util.EnumSet;

public class WatcherGoal extends Goal {
    private final GrinningStalkerEntity stalker;
    private PlayerEntity targetPlayer;
    private int gazeTicks = 0;
    private int soundCooldown = 200;
    private int torchCooldown = 120;
    private int maxLifetimeTicks = 1200; // 60 saniye

    public WatcherGoal(GrinningStalkerEntity stalker) {
        this.stalker = stalker;
        this.setControls(EnumSet.of(Control.MOVE, Control.LOOK));
    }

    @Override
    public boolean canStart() {
        if (stalker.getVariant() != StalkerVariant.WATCHER) {
            return false;
        }
        this.targetPlayer = stalker.getWorld().getClosestPlayer(stalker, 64.0);
        return this.targetPlayer != null && this.targetPlayer.isAlive();
    }

    @Override
    public boolean shouldContinue() {
        return stalker.getVariant() == StalkerVariant.WATCHER
                && targetPlayer != null
                && targetPlayer.isAlive()
                && maxLifetimeTicks > 0;
    }

    @Override
    public void tick() {
        maxLifetimeTicks--;
        if (targetPlayer == null) return;

        stalker.getLookControl().lookAt(targetPlayer, 30.0f, 30.0f);
        double distSq = stalker.squaredDistanceTo(targetPlayer);

        // Oyuncu 14 bloktan fazla yaklaşırsa veya yaratık hasar alırsa anında kaybolur
        if (distSq < 14.0 * 14.0 || stalker.hurtTime > 0) {
            stalker.triggerVanish();
            return;
        }

        // Oyuncunun doğrudan yaratığa bakıp bakmadığını kontrol et (dotProduct > 0.92)
        Vec3d playerLook = targetPlayer.getRotationVec(1.0f).normalize();
        Vec3d toStalker = stalker.getEyePos().subtract(targetPlayer.getEyePos()).normalize();
        double dot = playerLook.dotProduct(toStalker);

        if (dot > 0.92 && targetPlayer.canSee(stalker)) {
            gazeTicks++;
            if (gazeTicks == 5 && targetPlayer instanceof ServerPlayerEntity serverPlayer) {
                DreadManager.addEventDread(serverPlayer, DreadConfig.WATCHER_GAZE_DREAD_BONUS);
            }
            // Oyuncu 0.8 saniye boyunca doğrudan odaklanırsa görüş alanında kaybolur
            if (gazeTicks >= 16) {
                stalker.triggerVanish();
                return;
            }
        } else {
            gazeTicks = Math.max(0, gazeTicks - 1);
        }

        // Sahte sesler çıkarma (oyuncunun arkasında)
        if (--soundCooldown <= 0) {
            soundCooldown = 240 + stalker.getRandom().nextInt(160);
            if (targetPlayer instanceof ServerPlayerEntity serverPlayer) {
                ServerPlayNetworking.send(serverPlayer, new HallucinationEventS2CPayload(
                        HallucinationEventS2CPayload.EventType.FAKE_FOOTSTEP,
                        1.0f
                ));
            }
        }

        // Oyuncu bakmıyorken yakındaki meşaleleri söndürme
        if (--torchCooldown <= 0 && dot < 0.3) {
            torchCooldown = 140;
            extinguishNearbyTorch();
        }
    }

    private void extinguishNearbyTorch() {
        BlockPos origin = stalker.getBlockPos();
        for (BlockPos pos : BlockPos.iterateOutwards(origin, 14, 6, 14)) {
            BlockState state = stalker.getWorld().getBlockState(pos);
            if (state.isOf(Blocks.TORCH) || state.isOf(Blocks.WALL_TORCH)
                    || state.isOf(Blocks.SOUL_TORCH) || state.isOf(Blocks.SOUL_WALL_TORCH)) {
                stalker.getWorld().breakBlock(pos, true, stalker);
                stalker.getWorld().playSound(null, pos, ModSounds.AMBIENT_FAKE_TORCH_HISS, SoundCategory.HOSTILE, 0.9f, 0.8f);
                return;
            }
        }
    }
}
```

## Dosya: `src/main/java/com/studio/dread/item/EclipseOfTheHollowItem.java`
```java
package com.studio.dread.item;

import com.studio.dread.core.dread.DreadManager;
import com.studio.dread.registry.ModSounds;
import net.minecraft.entity.LivingEntity;
import net.minecraft.entity.effect.StatusEffectInstance;
import net.minecraft.entity.effect.StatusEffects;
import net.minecraft.entity.player.PlayerEntity;
import net.minecraft.item.ItemStack;
import net.minecraft.item.SwordItem;
import net.minecraft.item.ToolMaterials;
import net.minecraft.item.tooltip.TooltipType;
import net.minecraft.particle.ParticleTypes;
import net.minecraft.server.network.ServerPlayerEntity;
import net.minecraft.server.world.ServerWorld;
import net.minecraft.sound.SoundCategory;
import net.minecraft.text.Text;
import net.minecraft.util.Formatting;
import net.minecraft.util.Hand;
import net.minecraft.util.TypedActionResult;
import net.minecraft.util.math.Box;
import net.minecraft.util.math.Vec3d;
import net.minecraft.world.World;

import java.util.List;

public class EclipseOfTheHollowItem extends SwordItem {
    public EclipseOfTheHollowItem(Settings settings) {
        super(ToolMaterials.NETHERITE, settings);
    }

    @Override
    public TypedActionResult<ItemStack> use(World world, PlayerEntity user, Hand hand) {
        ItemStack stack = user.getStackInHand(hand);
        if (user.getItemCooldownManager().isCoolingDown(this)) {
            return TypedActionResult.fail(stack);
        }

        if (!world.isClient() && world instanceof ServerWorld serverWorld && user instanceof ServerPlayerEntity serverPlayer) {
            // Özel Yetenek: Umbral Cleave (12 blok ileriye gölge yarığı dalgası)
            Vec3d look = user.getRotationVec(1.0f).normalize();
            Vec3d origin = user.getEyePos();

            for (int step = 1; step <= 12; step++) {
                Vec3d point = origin.add(look.multiply(step));
                serverWorld.spawnParticles(ParticleTypes.SOUL_FIRE_FLAME, point.x, point.y, point.z, 8, 0.3, 0.3, 0.3, 0.02);
                serverWorld.spawnParticles(ParticleTypes.LARGE_SMOKE, point.x, point.y, point.z, 5, 0.25, 0.25, 0.25, 0.01);

                Box hitBox = new Box(point.x - 1.5, point.y - 1.5, point.z - 1.5, point.x + 1.5, point.y + 1.5, point.z + 1.5);
                List<LivingEntity> targets = serverWorld.getEntitiesByClass(
                        LivingEntity.class,
                        hitBox,
                        entity -> entity != user && entity.isAlive()
                );

                for (LivingEntity target : targets) {
                    target.damage(serverWorld.getDamageSources().magic(), 18.0f);
                    target.addStatusEffect(new StatusEffectInstance(StatusEffects.DARKNESS, 160, 0));
                    target.addStatusEffect(new StatusEffectInstance(StatusEffects.SLOWNESS, 100, 1));
                }
            }

            // Kullanıldığında oyuncunun Dread puanını 25 düşürür
            DreadManager.addEventDread(serverPlayer, -25.0f);
            serverWorld.playSound(null, user.getBlockPos(), ModSounds.STALKER_SLASH, SoundCategory.PLAYERS, 1.2f, 0.65f);
            user.getItemCooldownManager().set(this, 160); // 8 saniye cooldown
        }

        return TypedActionResult.success(stack, world.isClient());
    }

    @Override
    public void appendTooltip(ItemStack stack, TooltipContext context, List<Text> tooltip, TooltipType type) {
        tooltip.add(Text.translatable("item.dread.eclipse_of_the_hollow.desc1").formatted(Formatting.LIGHT_PURPLE, Formatting.BOLD));
        tooltip.add(Text.translatable("item.dread.eclipse_of_the_hollow.desc2").formatted(Formatting.GRAY));
        tooltip.add(Text.translatable("item.dread.eclipse_of_the_hollow.desc3").formatted(Formatting.GOLD));
    }
}
```

## Dosya: `src/main/java/com/studio/dread/item/GrinningHeartItem.java`
```java
package com.studio.dread.item;

import com.studio.dread.block.UmbralPortalBlock;
import com.studio.dread.registry.ModBlocks;
import com.studio.dread.registry.ModSounds;
import net.minecraft.entity.EquipmentSlot;
import net.minecraft.item.Item;
import net.minecraft.item.ItemStack;
import net.minecraft.item.ItemUsageContext;
import net.minecraft.item.tooltip.TooltipType;
import net.minecraft.sound.SoundCategory;
import net.minecraft.text.Text;
import net.minecraft.util.ActionResult;
import net.minecraft.util.Formatting;
import net.minecraft.util.math.BlockPos;
import net.minecraft.world.World;

import java.util.List;

public class GrinningHeartItem extends Item {
    public GrinningHeartItem(Settings settings) {
        super(settings);
    }

    @Override
    public ActionResult useOnBlock(ItemUsageContext context) {
        World world = context.getWorld();
        BlockPos clickedPos = context.getBlockPos();

        if (!world.getBlockState(clickedPos).isOf(ModBlocks.CHARRED_BONE_BLOCK)) {
            return ActionResult.PASS;
        }

        BlockPos insidePos = clickedPos.offset(context.getSide());
        if (!world.isClient()) {
            if (UmbralPortalBlock.tryLightPortal(world, insidePos)) {
                if (context.getPlayer() != null) {
                    context.getStack().damage(1, context.getPlayer(), EquipmentSlot.MAINHAND);
                }
                return ActionResult.SUCCESS;
            } else {
                world.playSound(null, clickedPos, ModSounds.STALKER_BONE_CLICK, SoundCategory.BLOCKS, 0.8f, 0.6f);
            }
        }
        return ActionResult.consume(world.isClient());
    }

    @Override
    public void appendTooltip(ItemStack stack, TooltipContext context, List<Text> tooltip, TooltipType type) {
        tooltip.add(Text.translatable("item.dread.grinning_heart.desc1").formatted(Formatting.GRAY));
        tooltip.add(Text.translatable("item.dread.grinning_heart.desc2").formatted(Formatting.DARK_RED));
    }
}
```

## Dosya: `src/main/java/com/studio/dread/item/LostJournalItem.java`
```java
package com.studio.dread.item;

import com.studio.dread.core.dread.DreadManager;
import com.studio.dread.core.dread.PlayerDreadState;
import com.studio.dread.core.persistence.DreadWorldPersistentState;
import com.studio.dread.registry.ModSounds;
import net.minecraft.entity.player.PlayerEntity;
import net.minecraft.item.Item;
import net.minecraft.item.ItemStack;
import net.minecraft.item.tooltip.TooltipType;
import net.minecraft.server.network.ServerPlayerEntity;
import net.minecraft.sound.SoundCategory;
import net.minecraft.text.Text;
import net.minecraft.util.Formatting;
import net.minecraft.util.Hand;
import net.minecraft.util.TypedActionResult;
import net.minecraft.world.World;

import java.util.List;

public class LostJournalItem extends Item {
    private final int chapterIndex; // 1..5

    public LostJournalItem(int chapterIndex, Settings settings) {
        super(settings);
        this.chapterIndex = chapterIndex;
    }

    public int getChapterIndex() {
        return chapterIndex;
    }

    @Override
    public TypedActionResult<ItemStack> use(World world, PlayerEntity user, Hand hand) {
        ItemStack stack = user.getStackInHand(hand);
        if (!world.isClient() && user instanceof ServerPlayerEntity serverPlayer) {
            DreadWorldPersistentState worldState = DreadWorldPersistentState.get(serverPlayer.getServer());
            PlayerDreadState state = worldState.getPlayerState(serverPlayer.getUuid());
            state.unlockJournal(chapterIndex);
            worldState.markDirty();
            DreadManager.forceSync(serverPlayer, state);

            serverPlayer.sendMessage(Text.translatable("journal.dread.chapter_" + chapterIndex + ".title")
                    .formatted(Formatting.DARK_RED, Formatting.BOLD), false);
            serverPlayer.sendMessage(Text.translatable("journal.dread.chapter_" + chapterIndex + ".body")
                    .formatted(Formatting.GRAY), false);

            world.playSound(null, user.getBlockPos(), ModSounds.STALKER_BONE_CLICK, SoundCategory.PLAYERS, 0.7f, 0.9f);
        }
        return TypedActionResult.success(stack, world.isClient());
    }

    @Override
    public void appendTooltip(ItemStack stack, TooltipContext context, List<Text> tooltip, TooltipType type) {
        tooltip.add(Text.translatable("journal.dread.chapter_" + chapterIndex + ".title").formatted(Formatting.GOLD));
        tooltip.add(Text.translatable("item.dread.lost_journal.hint").formatted(Formatting.DARK_GRAY));
    }
}
```

## Dosya: `src/main/java/com/studio/dread/item/TomeOfWhiteEyesItem.java`
```java
package com.studio.dread.item;

import com.studio.dread.registry.ModSounds;
import net.minecraft.entity.player.PlayerEntity;
import net.minecraft.item.Item;
import net.minecraft.item.ItemStack;
import net.minecraft.item.tooltip.TooltipType;
import net.minecraft.server.network.ServerPlayerEntity;
import net.minecraft.sound.SoundCategory;
import net.minecraft.text.Text;
import net.minecraft.util.Formatting;
import net.minecraft.util.Hand;
import net.minecraft.util.TypedActionResult;
import net.minecraft.world.World;

import java.util.List;

public class TomeOfWhiteEyesItem extends Item {
    public TomeOfWhiteEyesItem(Settings settings) {
        super(settings);
    }

    @Override
    public TypedActionResult<ItemStack> use(World world, PlayerEntity user, Hand hand) {
        ItemStack stack = user.getStackInHand(hand);
        if (!world.isClient() && user instanceof ServerPlayerEntity serverPlayer) {
            serverPlayer.sendMessage(Text.translatable("tome.dread.white_eyes.header")
                    .formatted(Formatting.WHITE, Formatting.BOLD), false);
            serverPlayer.sendMessage(Text.translatable("tome.dread.white_eyes.step1")
                    .formatted(Formatting.GOLD), false);
            serverPlayer.sendMessage(Text.translatable("tome.dread.white_eyes.step2")
                    .formatted(Formatting.RED), false);
            serverPlayer.sendMessage(Text.translatable("tome.dread.white_eyes.step3")
                    .formatted(Formatting.DARK_RED, Formatting.ITALIC), false);
            world.playSound(null, user.getBlockPos(), ModSounds.HEROBRINE_TOTEM_AWAKEN, SoundCategory.PLAYERS, 0.5f, 1.2f);
        }
        return TypedActionResult.success(stack, world.isClient());
    }

    @Override
    public void appendTooltip(ItemStack stack, TooltipContext context, List<Text> tooltip, TooltipType type) {
        tooltip.add(Text.translatable("item.dread.tome_of_white_eyes.desc1").formatted(Formatting.AQUA));
        tooltip.add(Text.translatable("item.dread.tome_of_white_eyes.desc2").formatted(Formatting.DARK_RED));
    }
}
```

## Dosya: `src/main/java/com/studio/dread/mixin/BackgroundRendererMixin.java`
```java
package com.studio.dread.mixin;

import com.mojang.blaze3d.systems.RenderSystem;
import com.studio.dread.config.DreadConfig;
import com.studio.dread.registry.ModDimensions;
import net.minecraft.client.MinecraftClient;
import net.minecraft.client.render.BackgroundRenderer;
import net.minecraft.client.render.Camera;
import org.spongepowered.asm.mixin.Mixin;
import org.spongepowered.asm.mixin.injection.At;
import org.spongepowered.asm.mixin.injection.Inject;
import org.spongepowered.asm.mixin.injection.callback.CallbackInfo;

@Mixin(BackgroundRenderer.class)
public abstract class BackgroundRendererMixin {

    /**
     * 4. Boyut (The Umbral Hollow): Maksimum 16 blok görüş mesafesi ve yoğun sis.
     */
    @Inject(method = "applyFog", at = @At("TAIL"))
    private static void dread$enforceUmbralHollow16BlockFog(Camera camera, BackgroundRenderer.FogType fogType,
                                                            float viewDistance, boolean thickFog, float tickDelta,
                                                            CallbackInfo ci) {
        MinecraftClient client = MinecraftClient.getInstance();
        if (client.world != null && client.world.getRegistryKey().equals(ModDimensions.UMBRAL_HOLLOW_WORLD_KEY)) {
            RenderSystem.setShaderFogStart(DreadConfig.UMBRAL_FOG_START);
            RenderSystem.setShaderFogEnd(DreadConfig.UMBRAL_FOG_END);
        }
    }
}
```

## Dosya: `src/main/java/com/studio/dread/mixin/LivingEntityDamageMixin.java`
```java
package com.studio.dread.mixin;

import com.studio.dread.entity.boss.HerobrineApparitionEntity;
import com.studio.dread.entity.stalker.GrinningStalkerEntity;
import com.studio.dread.entity.stalker.StalkerVariant;
import net.minecraft.entity.LivingEntity;
import net.minecraft.entity.damage.DamageSource;
import net.minecraft.entity.player.PlayerEntity;
import org.spongepowered.asm.mixin.Mixin;
import org.spongepowered.asm.mixin.injection.At;
import org.spongepowered.asm.mixin.injection.ModifyVariable;

@Mixin(LivingEntity.class)
public abstract class LivingEntityDamageMixin {

    /**
     * Kritik Hasar Kuralları:
     * 1. Herobrine Apparition (Boss öncesi): Oyuncuya ASLA hasar vermez (amount = 0.0f).
     * 2. Brink of Death (Versiyon 4): Oyuncuyu yarım kalpte (1.0 HP) bırakır, son darbeyi vurmaz, öldüremez.
     * 3. The Executioner (Versiyon 5): Dünya Hardcore ise oyuncuyu yarım kalpte (1.0 HP) bırakır; Hardcore değilse öldürebilir.
     */
    @ModifyVariable(method = "damage", at = @At("HEAD"), argsOnly = true, ordinal = 0)
    private float dread$clampStalkerAndHerobrineDamage(float amount, DamageSource source) {
        LivingEntity self = (LivingEntity) (Object) this;
        if (!(self instanceof PlayerEntity player)) {
            return amount;
        }

        if (source.getAttacker() instanceof HerobrineApparitionEntity) {
            return 0.0f;
        }

        if (source.getAttacker() instanceof GrinningStalkerEntity stalker) {
            StalkerVariant variant = stalker.getVariant();
            boolean isHardcore = player.getWorld().getLevelProperties().isHardcore();

            if (variant == StalkerVariant.WATCHER) {
                return 0.0f;
            }

            if (variant == StalkerVariant.BRINK_OF_DEATH
                    || (variant == StalkerVariant.EXECUTIONER && isHardcore)) {
                float currentHealth = player.getHealth();
                if (currentHealth <= 1.0f) {
                    return 0.0f;
                }
                if (currentHealth - amount < 1.0f) {
                    return Math.max(0.0f, currentHealth - 1.0f);
                }
            }
        }

        return amount;
    }
}
```

## Dosya: `src/main/java/com/studio/dread/mixin/ServerPlayerEntityMixin.java`
```java
package com.studio.dread.mixin;

import com.mojang.datafixers.util.Either;
import com.studio.dread.config.DreadConfig;
import com.studio.dread.core.dread.DreadManager;
import com.studio.dread.core.dread.PlayerDreadState;
import com.studio.dread.core.persistence.DreadWorldPersistentState;
import net.minecraft.entity.player.PlayerEntity;
import net.minecraft.server.network.ServerPlayerEntity;
import net.minecraft.text.Text;
import net.minecraft.util.Formatting;
import net.minecraft.util.Unit;
import net.minecraft.util.math.BlockPos;
import org.spongepowered.asm.mixin.Mixin;
import org.spongepowered.asm.mixin.injection.At;
import org.spongepowered.asm.mixin.injection.Inject;
import org.spongepowered.asm.mixin.injection.callback.CallbackInfo;
import org.spongepowered.asm.mixin.injection.callback.CallbackInfoReturnable;

@Mixin(ServerPlayerEntity.class)
public abstract class ServerPlayerEntityMixin {

    @Inject(method = "trySleep", at = @At("HEAD"), cancellable = true)
    private void dread$preventSleepWhenTerrified(BlockPos pos, CallbackInfoReturnable<Either<PlayerEntity.SleepFailureReason, Unit>> cir) {
        ServerPlayerEntity player = (ServerPlayerEntity) (Object) this;
        if (player.getServer() == null) return;

        DreadWorldPersistentState worldState = DreadWorldPersistentState.get(player.getServer());
        PlayerDreadState state = worldState.getPlayerState(player.getUuid());

        if (state.getDreadScore() >= DreadConfig.SLEEP_LOCK_DREAD) {
            player.sendMessage(Text.translatable("message.dread.cannot_sleep_terrified").formatted(Formatting.DARK_RED), true);
            cir.setReturnValue(Either.left(PlayerEntity.SleepFailureReason.NOT_SAFE));
        }
    }

    @Inject(method = "wakeUp", at = @At("TAIL"))
    private void dread$relieveDreadAfterFullSleep(boolean skipSleepTimer, boolean updateSleepingPlayers, CallbackInfo ci) {
        ServerPlayerEntity player = (ServerPlayerEntity) (Object) this;
        if (!skipSleepTimer && player.getServer() != null && player.getWorld().isDay()) {
            DreadManager.addEventDread(player, -DreadConfig.SLEEP_RELIEF_AMOUNT);
        }
    }

    @Inject(method = "copyFrom", at = @At("TAIL"))
    private void dread$retainFloorDreadOnRespawn(ServerPlayerEntity oldPlayer, boolean alive, CallbackInfo ci) {
        ServerPlayerEntity newPlayer = (ServerPlayerEntity) (Object) this;
        if (!alive && newPlayer.getServer() != null) {
            DreadWorldPersistentState worldState = DreadWorldPersistentState.get(newPlayer.getServer());
            PlayerDreadState state = worldState.getPlayerState(newPlayer.getUuid());
            state.setLockedAtMaxByBoss(false);
            if (state.getDreadScore() > DreadConfig.DEATH_FLOOR_DREAD) {
                state.setDreadScore(DreadConfig.DEATH_FLOOR_DREAD);
            }
            DreadManager.forceSync(newPlayer, state);
        }
    }
}
```

## Dosya: `src/main/java/com/studio/dread/network/DreadSyncS2CPayload.java`
```java
package com.studio.dread.network;

import com.studio.dread.config.DreadConfig;
import net.minecraft.network.RegistryByteBuf;
import net.minecraft.network.codec.PacketCodec;
import net.minecraft.network.codec.PacketCodecs;
import net.minecraft.network.packet.CustomPayload;
import net.minecraft.util.Identifier;

public record DreadSyncS2CPayload(
        float dreadScore,
        int readJournalsMask,
        int homeChunkX,
        int homeChunkZ
) implements CustomPayload {

    public static final CustomPayload.Id<DreadSyncS2CPayload> ID =
            new CustomPayload.Id<>(Identifier.of(DreadConfig.MOD_ID, "dread_sync"));

    public static final PacketCodec<RegistryByteBuf, DreadSyncS2CPayload> CODEC = PacketCodec.tuple(
            PacketCodecs.FLOAT, DreadSyncS2CPayload::dreadScore,
            PacketCodecs.VAR_INT, DreadSyncS2CPayload::readJournalsMask,
            PacketCodecs.INTEGER, DreadSyncS2CPayload::homeChunkX,
            PacketCodecs.INTEGER, DreadSyncS2CPayload::homeChunkZ,
            DreadSyncS2CPayload::new
    );

    @Override
    public Id<? extends CustomPayload> getId() {
        return ID;
    }
}
```

## Dosya: `src/main/java/com/studio/dread/network/HallucinationEventS2CPayload.java`
```java
package com.studio.dread.network;

import com.studio.dread.config.DreadConfig;
import net.minecraft.network.RegistryByteBuf;
import net.minecraft.network.codec.PacketCodec;
import net.minecraft.network.codec.PacketCodecs;
import net.minecraft.network.packet.CustomPayload;
import net.minecraft.util.Identifier;

public record HallucinationEventS2CPayload(
        int eventTypeOrdinal,
        float intensity
) implements CustomPayload {

    public enum EventType {
        FAKE_FOOTSTEP,
        FAKE_TORCH_FLICKER,
        STALKER_JUMPSCARE,
        HEROBRINE_FLASH,
        HEROBRINE_JUMPSCARE,
        ABYSSAL_GAZE_WARNING
    }

    public HallucinationEventS2CPayload(EventType type, float intensity) {
        this(type.ordinal(), intensity);
    }

    public EventType getEventType() {
        EventType[] values = EventType.values();
        return values[Math.max(0, Math.min(eventTypeOrdinal, values.length - 1))];
    }

    public static final CustomPayload.Id<HallucinationEventS2CPayload> ID =
            new CustomPayload.Id<>(Identifier.of(DreadConfig.MOD_ID, "hallucination_event"));

    public static final PacketCodec<RegistryByteBuf, HallucinationEventS2CPayload> CODEC = PacketCodec.tuple(
            PacketCodecs.VAR_INT, HallucinationEventS2CPayload::eventTypeOrdinal,
            PacketCodecs.FLOAT, HallucinationEventS2CPayload::intensity,
            HallucinationEventS2CPayload::new
    );

    @Override
    public Id<? extends CustomPayload> getId() {
        return ID;
    }
}
```

## Dosya: `src/main/java/com/studio/dread/network/ModPackets.java`
```java
package com.studio.dread.network;

import net.fabricmc.fabric.api.networking.v1.PayloadTypeRegistry;

public final class ModPackets {
    private ModPackets() {}

    public static void registerCommonPayloads() {
        PayloadTypeRegistry.playS2C().register(DreadSyncS2CPayload.ID, DreadSyncS2CPayload.CODEC);
        PayloadTypeRegistry.playS2C().register(HallucinationEventS2CPayload.ID, HallucinationEventS2CPayload.CODEC);
    }
}
```

## Dosya: `src/main/java/com/studio/dread/registry/ModAdvancementCriteria.java`
```java
package com.studio.dread.registry;

import com.mojang.serialization.Codec;
import com.mojang.serialization.codecs.RecordCodecBuilder;
import com.studio.dread.config.DreadConfig;
import net.minecraft.advancement.criterion.AbstractCriterion;
import net.minecraft.predicate.entity.EntityPredicate;
import net.minecraft.predicate.entity.LootContextPredicate;
import net.minecraft.registry.Registries;
import net.minecraft.registry.Registry;
import net.minecraft.server.network.ServerPlayerEntity;
import net.minecraft.util.Identifier;

import java.util.Optional;

public final class ModAdvancementCriteria {
    public static final SimpleDreadCriterion FIRST_DREAD_100 = register("first_dread_100");
    public static final SimpleDreadCriterion SURVIVE_HALF_HEART = register("survive_half_heart");
    public static final SimpleDreadCriterion ENTER_UMBRAL_HOLLOW = register("enter_umbral_hollow");
    public static final SimpleDreadCriterion SLAY_APEX_DREAD = register("slay_apex_dread");
    public static final SimpleDreadCriterion AWAKEN_HEROBRINE_TOTEM = register("awaken_herobrine_totem");
    public static final SimpleDreadCriterion SLAY_HEROBRINE_BOSS = register("slay_herobrine_boss");

    private ModAdvancementCriteria() {}

    private static SimpleDreadCriterion register(String name) {
        return Registry.register(
                Registries.CRITERION,
                Identifier.of(DreadConfig.MOD_ID, name),
                new SimpleDreadCriterion()
        );
    }

    public static void init() {}

    public static class SimpleDreadCriterion extends AbstractCriterion<SimpleDreadCriterion.Conditions> {
        @Override
        public Codec<Conditions> getConditionsCodec() {
            return Conditions.CODEC;
        }

        public void trigger(ServerPlayerEntity player) {
            this.trigger(player, conditions -> true);
        }

        public record Conditions(Optional<LootContextPredicate> player) implements AbstractCriterion.Conditions {
            public static final Codec<Conditions> CODEC = RecordCodecBuilder.create(instance ->
                    instance.group(
                            EntityPredicate.LOOT_CONTEXT_PREDICATE_CODEC.optionalFieldOf("player").forGetter(Conditions::player)
                    ).apply(instance, Conditions::new)
            );
        }
    }
}
```

## Dosya: `src/main/java/com/studio/dread/registry/ModBlocks.java`
```java
package com.studio.dread.registry;

import com.studio.dread.block.CharredBoneBlock;
import com.studio.dread.block.HerobrineTotemBlock;
import com.studio.dread.block.UmbralPortalBlock;
import com.studio.dread.block.WatchingBarkBlock;
import com.studio.dread.config.DreadConfig;
import net.minecraft.block.AbstractBlock;
import net.minecraft.block.Block;
import net.minecraft.block.FlowerBlock;
import net.minecraft.block.MapColor;
import net.minecraft.block.PillarBlock;
import net.minecraft.block.SnowBlock;
import net.minecraft.entity.effect.StatusEffects;
import net.minecraft.item.BlockItem;
import net.minecraft.item.Item;
import net.minecraft.registry.Registries;
import net.minecraft.registry.Registry;
import net.minecraft.sound.BlockSoundGroup;
import net.minecraft.util.Identifier;

public final class ModBlocks {
    public static final Block CHARRED_BONE_BLOCK = registerWithItem("charred_bone_block",
            new CharredBoneBlock(AbstractBlock.Settings.create()
                    .mapColor(MapColor.BLACK)
                    .strength(3.0f, 9.0f)
                    .requiresTool()
                    .sounds(BlockSoundGroup.BONE)));

    public static final Block UMBRAL_PORTAL_BLOCK = registerWithoutItem("umbral_portal_block",
            new UmbralPortalBlock(AbstractBlock.Settings.create()
                    .mapColor(MapColor.BLACK)
                    .noCollision()
                    .strength(-1.0f)
                    .luminance(state -> 5)
                    .sounds(BlockSoundGroup.GLASS)));

    public static final Block ASH_LAYER = registerWithItem("ash_layer",
            new SnowBlock(AbstractBlock.Settings.create()
                    .mapColor(MapColor.GRAY)
                    .strength(0.2f)
                    .sounds(BlockSoundGroup.SAND)));

    public static final Block ASHEN_SOIL = registerWithItem("ashen_soil",
            new Block(AbstractBlock.Settings.create()
                    .mapColor(MapColor.DEEPSLATE_GRAY)
                    .strength(0.8f)
                    .sounds(BlockSoundGroup.GRAVEL)));

    public static final Block UMBRAL_STONE = registerWithItem("umbral_stone",
            new Block(AbstractBlock.Settings.create()
                    .mapColor(MapColor.BLACK)
                    .strength(2.5f, 8.0f)
                    .requiresTool()
                    .sounds(BlockSoundGroup.DEEPSLATE)));

    public static final Block PETRIFIED_LOG = registerWithItem("petrified_log",
            new PillarBlock(AbstractBlock.Settings.create()
                    .mapColor(MapColor.STONE_GRAY)
                    .strength(2.2f, 5.0f)
                    .requiresTool()
                    .sounds(BlockSoundGroup.BASALT)));

    public static final Block WATCHING_BARK = registerWithItem("watching_bark",
            new WatchingBarkBlock(AbstractBlock.Settings.create()
                    .mapColor(MapColor.STONE_GRAY)
                    .strength(2.5f, 6.0f)
                    .sounds(BlockSoundGroup.BASALT)));

    public static final Block ASH_BLOOM = registerWithItem("ash_bloom",
            new FlowerBlock(StatusEffects.DARKNESS, 6.0f, AbstractBlock.Settings.create()
                    .mapColor(MapColor.LIGHT_GRAY)
                    .noCollision()
                    .breakInstantly()
                    .luminance(state -> 4)
                    .sounds(BlockSoundGroup.GRASS)));

    public static final Block UMBRAL_OBELISK_BLOCK = registerWithItem("umbral_obelisk_block",
            new Block(AbstractBlock.Settings.create()
                    .mapColor(MapColor.BLACK)
                    .strength(-1.0f, 3600000.0f)
                    .dropsNothing()
                    .sounds(BlockSoundGroup.NETHERITE)));

    public static final Block HEROBRINE_TOTEM_BLOCK = registerWithItem("herobrine_totem_block",
            new HerobrineTotemBlock(AbstractBlock.Settings.create()
                    .mapColor(MapColor.GOLD)
                    .strength(4.0f, 12.0f)
                    .requiresTool()
                    .luminance(state -> 7)
                    .sounds(BlockSoundGroup.METAL)));

    private ModBlocks() {}

    private static Block registerWithItem(String name, Block block) {
        Identifier id = Identifier.of(DreadConfig.MOD_ID, name);
        Block registered = Registry.register(Registries.BLOCK, id, block);
        Registry.register(Registries.ITEM, id, new BlockItem(registered, new Item.Settings()));
        return registered;
    }

    private static Block registerWithoutItem(String name, Block block) {
        Identifier id = Identifier.of(DreadConfig.MOD_ID, name);
        return Registry.register(Registries.BLOCK, id, block);
    }

    public static void init() {}
}
```

## Dosya: `src/main/java/com/studio/dread/registry/ModDimensions.java`
```java
package com.studio.dread.registry;

import com.studio.dread.config.DreadConfig;
import net.minecraft.registry.RegistryKey;
import net.minecraft.registry.RegistryKeys;
import net.minecraft.util.Identifier;
import net.minecraft.world.World;
import net.minecraft.world.dimension.DimensionOptions;
import net.minecraft.world.dimension.DimensionType;

public final class ModDimensions {
    public static final Identifier UMBRAL_HOLLOW_ID = Identifier.of(DreadConfig.MOD_ID, "umbral_hollow");

    public static final RegistryKey<DimensionOptions> UMBRAL_HOLLOW_DIMENSION_KEY = RegistryKey.of(
            RegistryKeys.DIMENSION,
            UMBRAL_HOLLOW_ID
    );

    public static final RegistryKey<World> UMBRAL_HOLLOW_WORLD_KEY = RegistryKey.of(
            RegistryKeys.WORLD,
            UMBRAL_HOLLOW_ID
    );

    public static final RegistryKey<DimensionType> UMBRAL_HOLLOW_TYPE_KEY = RegistryKey.of(
            RegistryKeys.DIMENSION_TYPE,
            Identifier.of(DreadConfig.MOD_ID, "umbral_hollow_type")
    );

    private ModDimensions() {}

    public static void init() {}
}
```

## Dosya: `src/main/java/com/studio/dread/registry/ModItems.java`
```java
package com.studio.dread.registry;

import com.studio.dread.config.DreadConfig;
import com.studio.dread.item.EclipseOfTheHollowItem;
import com.studio.dread.item.GrinningHeartItem;
import com.studio.dread.item.LostJournalItem;
import com.studio.dread.item.TomeOfWhiteEyesItem;
import net.fabricmc.fabric.api.itemgroup.v1.FabricItemGroup;
import net.minecraft.item.Item;
import net.minecraft.item.ItemGroup;
import net.minecraft.item.ItemStack;
import net.minecraft.item.SwordItem;
import net.minecraft.item.ToolMaterials;
import net.minecraft.registry.Registries;
import net.minecraft.registry.Registry;
import net.minecraft.text.Text;
import net.minecraft.util.Identifier;
import net.minecraft.util.Rarity;

public final class ModItems {
    public static final Item CHARRED_BONE_SHARD = register("charred_bone_shard",
            new Item(new Item.Settings()));

    public static final Item GRINNING_HEART = register("grinning_heart",
            new GrinningHeartItem(new Item.Settings().maxDamage(3).rarity(Rarity.RARE)));

    public static final Item UMBRAL_CORE = register("umbral_core",
            new Item(new Item.Settings().maxCount(16).rarity(Rarity.EPIC).fireproof()));

    public static final Item LOST_JOURNAL_1 = register("lost_journal_1",
            new LostJournalItem(1, new Item.Settings().maxCount(1).rarity(Rarity.UNCOMMON)));
    public static final Item LOST_JOURNAL_2 = register("lost_journal_2",
            new LostJournalItem(2, new Item.Settings().maxCount(1).rarity(Rarity.UNCOMMON)));
    public static final Item LOST_JOURNAL_3 = register("lost_journal_3",
            new LostJournalItem(3, new Item.Settings().maxCount(1).rarity(Rarity.UNCOMMON)));
    public static final Item LOST_JOURNAL_4 = register("lost_journal_4",
            new LostJournalItem(4, new Item.Settings().maxCount(1).rarity(Rarity.UNCOMMON)));
    public static final Item LOST_JOURNAL_5 = register("lost_journal_5",
            new LostJournalItem(5, new Item.Settings().maxCount(1).rarity(Rarity.UNCOMMON)));

    public static final Item TOME_OF_WHITE_EYES = register("tome_of_white_eyes",
            new TomeOfWhiteEyesItem(new Item.Settings().maxCount(1).rarity(Rarity.EPIC).fireproof()));

    public static final Item ECLIPSE_OF_THE_HOLLOW = register("eclipse_of_the_hollow",
            new EclipseOfTheHollowItem(new Item.Settings()
                    .attributeModifiers(SwordItem.createAttributeModifiers(ToolMaterials.NETHERITE, 7, -2.2f))
                    .maxCount(1)
                    .rarity(Rarity.EPIC)
                    .fireproof()));

    public static final ItemGroup DREAD_ITEM_GROUP = Registry.register(
            Registries.ITEM_GROUP,
            Identifier.of(DreadConfig.MOD_ID, "dread_group"),
            FabricItemGroup.builder()
                    .icon(() -> new ItemStack(GRINNING_HEART))
                    .displayName(Text.translatable("itemGroup.dread.dread_group"))
                    .entries((context, entries) -> {
                        entries.add(ModBlocks.CHARRED_BONE_BLOCK);
                        entries.add(ModBlocks.ASH_LAYER);
                        entries.add(ModBlocks.ASHEN_SOIL);
                        entries.add(ModBlocks.UMBRAL_STONE);
                        entries.add(ModBlocks.PETRIFIED_LOG);
                        entries.add(ModBlocks.WATCHING_BARK);
                        entries.add(ModBlocks.ASH_BLOOM);
                        entries.add(ModBlocks.UMBRAL_OBELISK_BLOCK);
                        entries.add(ModBlocks.HEROBRINE_TOTEM_BLOCK);
                        entries.add(CHARRED_BONE_SHARD);
                        entries.add(GRINNING_HEART);
                        entries.add(UMBRAL_CORE);
                        entries.add(LOST_JOURNAL_1);
                        entries.add(LOST_JOURNAL_2);
                        entries.add(LOST_JOURNAL_3);
                        entries.add(LOST_JOURNAL_4);
                        entries.add(LOST_JOURNAL_5);
                        entries.add(TOME_OF_WHITE_EYES);
                        entries.add(ECLIPSE_OF_THE_HOLLOW);
                    })
                    .build()
    );

    private ModItems() {}

    public static Item getJournalByIndex(int index1To5) {
        return switch (index1To5) {
            case 1 -> LOST_JOURNAL_1;
            case 2 -> LOST_JOURNAL_2;
            case 3 -> LOST_JOURNAL_3;
            case 4 -> LOST_JOURNAL_4;
            default -> LOST_JOURNAL_5;
        };
    }

    private static Item register(String name, Item item) {
        return Registry.register(Registries.ITEM, Identifier.of(DreadConfig.MOD_ID, name), item);
    }

    public static void init() {}
}
```

## Dosya: `src/main/java/com/studio/dread/registry/ModSounds.java`
```java
package com.studio.dread.registry;

import com.studio.dread.config.DreadConfig;
import net.minecraft.registry.Registries;
import net.minecraft.registry.Registry;
import net.minecraft.sound.SoundEvent;
import net.minecraft.util.Identifier;

public final class ModSounds {
    public static final SoundEvent AMBIENT_DREAD_DRONE = register("ambient.dread_drone");
    public static final SoundEvent AMBIENT_UMBRAL_WIND = register("ambient.umbral_wind");
    public static final SoundEvent AMBIENT_FAKE_FOOTSTEP = register("ambient.fake_footstep");
    public static final SoundEvent AMBIENT_FAKE_TORCH_HISS = register("ambient.fake_torch_hiss");
    public static final SoundEvent AMBIENT_HEARTBEAT = register("ambient.heartbeat");

    public static final SoundEvent STALKER_FOOTSTEP_HEAVY = register("entity.stalker.footstep_heavy");
    public static final SoundEvent STALKER_BONE_CLICK = register("entity.stalker.bone_click");
    public static final SoundEvent STALKER_BREATHING = register("entity.stalker.breathing");
    public static final SoundEvent STALKER_SLASH = register("entity.stalker.slash");
    public static final SoundEvent STALKER_VANISH = register("entity.stalker.vanish");
    public static final SoundEvent STALKER_JUMPSCARE = register("event.jumpscare.stalker");

    public static final SoundEvent PITCH_GRIN_CRAWL = register("entity.pitch_grin.crawl");
    public static final SoundEvent PITCH_GRIN_SHRIEK_BITE = register("entity.pitch_grin.shriek_bite");

    public static final SoundEvent APEX_DREAD_ROAR = register("entity.apex_dread.roar");
    public static final SoundEvent APEX_DREAD_GAZE_CHARGE = register("entity.apex_dread.gaze_charge");
    public static final SoundEvent MUSIC_BOSS_APEX_DREAD = register("music.boss.apex_dread");

    public static final SoundEvent HEROBRINE_JUMPSCARE_STATIC = register("event.jumpscare.herobrine_static");
    public static final SoundEvent HEROBRINE_TOTEM_AWAKEN = register("event.herobrine.totem_awaken");
    public static final SoundEvent MUSIC_BOSS_HEROBRINE_P1 = register("music.boss.herobrine_phase1");
    public static final SoundEvent MUSIC_BOSS_HEROBRINE_P2 = register("music.boss.herobrine_phase2");
    public static final SoundEvent MUSIC_BOSS_HEROBRINE_P3 = register("music.boss.herobrine_phase3");

    private ModSounds() {}

    private static SoundEvent register(String path) {
        Identifier id = Identifier.of(DreadConfig.MOD_ID, path);
        return Registry.register(Registries.SOUND_EVENT, id, SoundEvent.of(id));
    }

    public static void init() {
        // Sınıf yüklemesini tetikler
    }
}
```

## Dosya: `src/main/resources/assets/dread/animations/dread_entities.animation.json`
```json
{
  "format_version": "1.8.0",
  "animations": {
    "animation.dread.idle": {
      "loop": true,
      "animation_length": 3.0,
      "bones": {
        "upper_torso": {
          "rotation": {
            "0.0": [
              4.0,
              0.0,
              0.0
            ],
            "1.5": [
              7.5,
              0.0,
              1.5
            ],
            "3.0": [
              4.0,
              0.0,
              0.0
            ]
          }
        },
        "head": {
          "rotation": {
            "0.0": [
              0.0,
              0.0,
              -10.0
            ],
            "1.2": [
              2.0,
              0.0,
              -10.0
            ],
            "1.35": [
              -4.0,
              6.0,
              14.0
            ],
            "3.0": [
              0.0,
              0.0,
              -10.0
            ]
          }
        },
        "left_fingers": {
          "rotation": {
            "0.0": [
              0.0,
              0.0,
              0.0
            ],
            "0.75": [
              18.0,
              0.0,
              0.0
            ],
            "1.5": [
              0.0,
              0.0,
              0.0
            ]
          }
        },
        "right_fingers": {
          "rotation": {
            "0.0": [
              12.0,
              0.0,
              0.0
            ],
            "1.5": [
              -8.0,
              0.0,
              0.0
            ],
            "3.0": [
              12.0,
              0.0,
              0.0
            ]
          }
        }
      }
    },
    "animation.dread.watching": {
      "loop": true,
      "animation_length": 4.0,
      "bones": {
        "upper_torso": {
          "rotation": {
            "0.0": [
              3.0,
              0.0,
              0.0
            ],
            "2.0": [
              5.0,
              0.0,
              0.0
            ],
            "4.0": [
              3.0,
              0.0,
              0.0
            ]
          }
        },
        "head": {
          "rotation": {
            "0.0": [
              0.0,
              0.0,
              15.0
            ],
            "2.0": [
              0.0,
              0.0,
              -15.0
            ],
            "4.0": [
              0.0,
              0.0,
              15.0
            ]
          }
        }
      }
    },
    "animation.dread.walk": {
      "loop": true,
      "animation_length": 1.6,
      "bones": {
        "upper_torso": {
          "rotation": {
            "0.0": [
              18.0,
              -4.0,
              0.0
            ],
            "0.8": [
              20.0,
              4.0,
              0.0
            ],
            "1.6": [
              18.0,
              -4.0,
              0.0
            ]
          }
        },
        "left_thigh": {
          "rotation": {
            "0.0": [
              -28.0,
              0.0,
              0.0
            ],
            "0.8": [
              28.0,
              0.0,
              0.0
            ],
            "1.6": [
              -28.0,
              0.0,
              0.0
            ]
          }
        },
        "right_thigh": {
          "rotation": {
            "0.0": [
              28.0,
              0.0,
              0.0
            ],
            "0.8": [
              -28.0,
              0.0,
              0.0
            ],
            "1.6": [
              28.0,
              0.0,
              0.0
            ]
          }
        },
        "left_upper_arm": {
          "rotation": {
            "0.0": [
              22.0,
              0.0,
              -4.0
            ],
            "0.8": [
              -22.0,
              0.0,
              -4.0
            ],
            "1.6": [
              22.0,
              0.0,
              -4.0
            ]
          }
        },
        "right_upper_arm": {
          "rotation": {
            "0.0": [
              -22.0,
              0.0,
              4.0
            ],
            "0.8": [
              22.0,
              0.0,
              4.0
            ],
            "1.6": [
              -22.0,
              0.0,
              4.0
            ]
          }
        }
      }
    },
    "animation.dread.sprint": {
      "loop": true,
      "animation_length": 0.65,
      "bones": {
        "pelvis": {
          "position": {
            "0.0": [
              0.0,
              -18.0,
              0.0
            ],
            "0.32": [
              0.0,
              -15.0,
              0.0
            ],
            "0.65": [
              0.0,
              -18.0,
              0.0
            ]
          }
        },
        "lower_torso": {
          "rotation": [
            78.0,
            0.0,
            0.0
          ]
        },
        "head": {
          "rotation": [
            -68.0,
            0.0,
            0.0
          ]
        },
        "left_upper_arm": {
          "rotation": {
            "0.0": [
              -75.0,
              0.0,
              -12.0
            ],
            "0.32": [
              35.0,
              0.0,
              -12.0
            ],
            "0.65": [
              -75.0,
              0.0,
              -12.0
            ]
          }
        },
        "right_upper_arm": {
          "rotation": {
            "0.0": [
              35.0,
              0.0,
              12.0
            ],
            "0.32": [
              -75.0,
              0.0,
              12.0
            ],
            "0.65": [
              35.0,
              0.0,
              12.0
            ]
          }
        },
        "left_thigh": {
          "rotation": {
            "0.0": [
              45.0,
              0.0,
              0.0
            ],
            "0.32": [
              -55.0,
              0.0,
              0.0
            ],
            "0.65": [
              45.0,
              0.0,
              0.0
            ]
          }
        },
        "right_thigh": {
          "rotation": {
            "0.0": [
              -55.0,
              0.0,
              0.0
            ],
            "0.32": [
              45.0,
              0.0,
              0.0
            ],
            "0.65": [
              -55.0,
              0.0,
              0.0
            ]
          }
        }
      }
    },
    "animation.dread.crawl": {
      "loop": true,
      "animation_length": 1.1,
      "bones": {
        "pelvis": {
          "position": {
            "0.0": [
              0.0,
              -21.0,
              0.0
            ],
            "0.55": [
              0.0,
              -20.0,
              0.0
            ],
            "1.1": [
              0.0,
              -21.0,
              0.0
            ]
          },
          "rotation": {
            "0.0": [
              0.0,
              -12.0,
              0.0
            ],
            "0.55": [
              0.0,
              12.0,
              0.0
            ],
            "1.1": [
              0.0,
              -12.0,
              0.0
            ]
          }
        },
        "lower_torso": {
          "rotation": {
            "0.0": [
              84.0,
              16.0,
              -6.0
            ],
            "0.55": [
              84.0,
              -16.0,
              6.0
            ],
            "1.1": [
              84.0,
              16.0,
              -6.0
            ]
          }
        },
        "upper_torso": {
          "rotation": {
            "0.0": [
              4.0,
              -20.0,
              8.0
            ],
            "0.55": [
              4.0,
              20.0,
              -8.0
            ],
            "1.1": [
              4.0,
              -20.0,
              8.0
            ]
          }
        },
        "head": {
          "rotation": {
            "0.0": [
              -82.0,
              8.0,
              5.0
            ],
            "0.55": [
              -82.0,
              -8.0,
              -5.0
            ],
            "1.1": [
              -82.0,
              8.0,
              5.0
            ]
          }
        },
        "left_upper_arm": {
          "rotation": {
            "0.0": [
              -85.0,
              -25.0,
              -35.0
            ],
            "0.55": [
              15.0,
              10.0,
              -25.0
            ],
            "1.1": [
              -85.0,
              -25.0,
              -35.0
            ]
          }
        },
        "left_forearm": {
          "rotation": {
            "0.0": [
              -45.0,
              0.0,
              0.0
            ],
            "0.55": [
              -95.0,
              0.0,
              0.0
            ],
            "1.1": [
              -45.0,
              0.0,
              0.0
            ]
          }
        },
        "right_upper_arm": {
          "rotation": {
            "0.0": [
              15.0,
              -10.0,
              25.0
            ],
            "0.55": [
              -85.0,
              25.0,
              35.0
            ],
            "1.1": [
              15.0,
              -10.0,
              25.0
            ]
          }
        },
        "right_forearm": {
          "rotation": {
            "0.0": [
              -95.0,
              0.0,
              0.0
            ],
            "0.55": [
              -45.0,
              0.0,
              0.0
            ],
            "1.1": [
              -95.0,
              0.0,
              0.0
            ]
          }
        },
        "left_thigh": {
          "rotation": {
            "0.0": [
              40.0,
              -30.0,
              0.0
            ],
            "0.55": [
              -60.0,
              -20.0,
              0.0
            ],
            "1.1": [
              40.0,
              -30.0,
              0.0
            ]
          }
        },
        "right_thigh": {
          "rotation": {
            "0.0": [
              -60.0,
              20.0,
              0.0
            ],
            "0.55": [
              40.0,
              30.0,
              0.0
            ],
            "1.1": [
              -60.0,
              20.0,
              0.0
            ]
          }
        }
      }
    },
    "animation.dread.climb": {
      "loop": true,
      "animation_length": 0.8,
      "bones": {
        "upper_torso": {
          "rotation": [
            -15.0,
            0.0,
            0.0
          ]
        },
        "left_upper_arm": {
          "rotation": {
            "0.0": [
              -140.0,
              0.0,
              0.0
            ],
            "0.4": [
              -45.0,
              0.0,
              0.0
            ],
            "0.8": [
              -140.0,
              0.0,
              0.0
            ]
          }
        },
        "right_upper_arm": {
          "rotation": {
            "0.0": [
              -45.0,
              0.0,
              0.0
            ],
            "0.4": [
              -140.0,
              0.0,
              0.0
            ],
            "0.8": [
              -45.0,
              0.0,
              0.0
            ]
          }
        },
        "left_thigh": {
          "rotation": {
            "0.0": [
              -65.0,
              0.0,
              0.0
            ],
            "0.4": [
              15.0,
              0.0,
              0.0
            ],
            "0.8": [
              -65.0,
              0.0,
              0.0
            ]
          }
        },
        "right_thigh": {
          "rotation": {
            "0.0": [
              15.0,
              0.0,
              0.0
            ],
            "0.4": [
              -65.0,
              0.0,
              0.0
            ],
            "0.8": [
              15.0,
              0.0,
              0.0
            ]
          }
        }
      }
    },
    "animation.dread.observe": {
      "loop": "hold_on_last_frame",
      "animation_length": 2.2,
      "bones": {
        "pelvis": {
          "position": {
            "0.0": [
              0.0,
              0.0,
              0.0
            ],
            "0.6": [
              0.0,
              -16.0,
              0.0
            ]
          }
        },
        "upper_torso": {
          "rotation": {
            "0.0": [
              10.0,
              0.0,
              0.0
            ],
            "0.6": [
              42.0,
              0.0,
              0.0
            ]
          }
        },
        "head": {
          "rotation": {
            "0.0": [
              0.0,
              0.0,
              0.0
            ],
            "0.8": [
              -25.0,
              0.0,
              38.0
            ]
          }
        },
        "lower_jaw": {
          "rotation": {
            "0.0": [
              0.0,
              0.0,
              0.0
            ],
            "1.0": [
              28.0,
              0.0,
              0.0
            ]
          }
        }
      }
    },
    "animation.dread.attack": {
      "loop": false,
      "animation_length": 0.55,
      "bones": {
        "upper_torso": {
          "rotation": {
            "0.0": [
              0.0,
              0.0,
              0.0
            ],
            "0.18": [
              -22.0,
              0.0,
              0.0
            ],
            "0.35": [
              35.0,
              0.0,
              0.0
            ],
            "0.55": [
              0.0,
              0.0,
              0.0
            ]
          }
        },
        "left_upper_arm": {
          "rotation": {
            "0.0": [
              0.0,
              0.0,
              0.0
            ],
            "0.18": [
              -135.0,
              -20.0,
              0.0
            ],
            "0.35": [
              -25.0,
              30.0,
              0.0
            ],
            "0.55": [
              0.0,
              0.0,
              0.0
            ]
          }
        },
        "right_upper_arm": {
          "rotation": {
            "0.0": [
              0.0,
              0.0,
              0.0
            ],
            "0.18": [
              -135.0,
              20.0,
              0.0
            ],
            "0.35": [
              -25.0,
              -30.0,
              0.0
            ],
            "0.55": [
              0.0,
              0.0,
              0.0
            ]
          }
        },
        "lower_jaw": {
          "rotation": {
            "0.0": [
              0.0,
              0.0,
              0.0
            ],
            "0.2": [
              35.0,
              0.0,
              0.0
            ],
            "0.55": [
              0.0,
              0.0,
              0.0
            ]
          }
        }
      }
    },
    "animation.dread.vanish": {
      "loop": false,
      "animation_length": 0.4,
      "bones": {
        "root": {
          "scale": {
            "0.0": [
              1.0,
              1.0,
              1.0
            ],
            "0.25": [
              1.15,
              1.25,
              1.15
            ],
            "0.4": [
              0.0,
              0.0,
              0.0
            ]
          }
        }
      }
    },
    "animation.dread.jumpscare": {
      "loop": false,
      "animation_length": 0.85,
      "bones": {
        "root": {
          "position": {
            "0.0": [
              0.0,
              0.0,
              12.0
            ],
            "0.15": [
              0.0,
              -4.0,
              -6.0
            ],
            "0.85": [
              0.0,
              -4.0,
              -6.0
            ]
          }
        },
        "head": {
          "rotation": {
            "0.0": [
              0.0,
              0.0,
              -25.0
            ],
            "0.2": [
              0.0,
              0.0,
              25.0
            ],
            "0.4": [
              0.0,
              0.0,
              -20.0
            ],
            "0.85": [
              0.0,
              0.0,
              0.0
            ]
          }
        },
        "lower_jaw": {
          "rotation": [
            42.0,
            0.0,
            0.0
          ]
        }
      }
    },
    "animation.dread.damaged": {
      "loop": false,
      "animation_length": 0.35,
      "bones": {
        "upper_torso": {
          "rotation": {
            "0.0": [
              0.0,
              0.0,
              0.0
            ],
            "0.12": [
              -24.0,
              0.0,
              8.0
            ],
            "0.35": [
              0.0,
              0.0,
              0.0
            ]
          }
        }
      }
    },
    "animation.dread.death": {
      "loop": "hold_on_last_frame",
      "animation_length": 2.8,
      "bones": {
        "root": {
          "rotation": {
            "0.0": [
              0.0,
              0.0,
              0.0
            ],
            "1.2": [
              -88.0,
              0.0,
              0.0
            ]
          },
          "scale": {
            "0.0": [
              1.0,
              1.0,
              1.0
            ],
            "2.0": [
              1.0,
              1.0,
              1.0
            ],
            "2.8": [
              0.0,
              0.0,
              0.0
            ]
          }
        }
      }
    },
    "animation.dread.boss_phase_1_to_2": {
      "loop": false,
      "animation_length": 2.5,
      "bones": {
        "upper_torso": {
          "rotation": {
            "0.0": [
              0.0,
              0.0,
              0.0
            ],
            "0.8": [
              -35.0,
              0.0,
              0.0
            ],
            "2.5": [
              0.0,
              0.0,
              0.0
            ]
          }
        },
        "left_upper_arm": {
          "rotation": {
            "0.0": [
              0.0,
              0.0,
              0.0
            ],
            "0.8": [
              -110.0,
              0.0,
              -45.0
            ],
            "2.5": [
              0.0,
              0.0,
              0.0
            ]
          }
        },
        "right_upper_arm": {
          "rotation": {
            "0.0": [
              0.0,
              0.0,
              0.0
            ],
            "0.8": [
              -110.0,
              0.0,
              45.0
            ],
            "2.5": [
              0.0,
              0.0,
              0.0
            ]
          }
        }
      }
    },
    "animation.dread.boss_phase_2_to_3": {
      "loop": false,
      "animation_length": 2.5,
      "bones": {
        "root": {
          "position": {
            "0.0": [
              0.0,
              0.0,
              0.0
            ],
            "1.0": [
              0.0,
              16.0,
              0.0
            ],
            "2.5": [
              0.0,
              0.0,
              0.0
            ]
          }
        },
        "lower_jaw": {
          "rotation": {
            "0.0": [
              0.0,
              0.0,
              0.0
            ],
            "1.0": [
              45.0,
              0.0,
              0.0
            ],
            "2.5": [
              20.0,
              0.0,
              0.0
            ]
          }
        }
      }
    }
  }
}
```

## Dosya: `src/main/resources/assets/dread/geo/apex_dread.geo.json`
```json
{
  "format_version": "1.12.0",
  "minecraft:geometry": [
    {
      "description": {
        "identifier": "geometry.dread.grinning_stalker",
        "texture_width": 128,
        "texture_height": 128,
        "visible_bounds_width": 3.0,
        "visible_bounds_height": 4.5,
        "visible_bounds_offset": [
          0,
          1.75,
          0
        ]
      },
      "bones": [
        {
          "name": "root",
          "pivot": [
            0,
            0,
            0
          ]
        },
        {
          "name": "pelvis",
          "parent": "root",
          "pivot": [
            0,
            26,
            0
          ],
          "cubes": [
            {
              "origin": [
                -4.5,
                24,
                -2.5
              ],
              "size": [
                9,
                5,
                5
              ],
              "uv": [
                0,
                28
              ]
            }
          ]
        },
        {
          "name": "lower_torso",
          "parent": "pelvis",
          "pivot": [
            0,
            29,
            0
          ],
          "cubes": [
            {
              "origin": [
                -4,
                29,
                -2
              ],
              "size": [
                8,
                10,
                4
              ],
              "uv": [
                28,
                28
              ]
            }
          ]
        },
        {
          "name": "upper_torso",
          "parent": "lower_torso",
          "pivot": [
            0,
            39,
            0
          ],
          "cubes": [
            {
              "origin": [
                -6,
                39,
                -3
              ],
              "size": [
                12,
                12,
                6
              ],
              "uv": [
                0,
                10
              ]
            }
          ]
        },
        {
          "name": "neck",
          "parent": "upper_torso",
          "pivot": [
            0,
            51,
            0
          ],
          "cubes": [
            {
              "origin": [
                -2,
                50,
                -2
              ],
              "size": [
                4,
                4,
                4
              ],
              "uv": [
                48,
                0
              ]
            }
          ]
        },
        {
          "name": "head",
          "parent": "neck",
          "pivot": [
            0,
            54,
            0
          ],
          "cubes": [
            {
              "origin": [
                -4.5,
                53,
                -4.5
              ],
              "size": [
                9,
                9,
                9
              ],
              "uv": [
                0,
                0
              ]
            }
          ]
        },
        {
          "name": "lower_jaw",
          "parent": "head",
          "pivot": [
            0,
            54,
            -1
          ],
          "cubes": [
            {
              "origin": [
                -4.5,
                51,
                -4.5
              ],
              "size": [
                9,
                3,
                9
              ],
              "uv": [
                36,
                10
              ]
            }
          ]
        },
        {
          "name": "left_upper_arm",
          "parent": "upper_torso",
          "pivot": [
            -6,
            49,
            0
          ],
          "cubes": [
            {
              "origin": [
                -9,
                31,
                -1.5
              ],
              "size": [
                3,
                19,
                3
              ],
              "uv": [
                52,
                28
              ]
            }
          ]
        },
        {
          "name": "left_forearm",
          "parent": "left_upper_arm",
          "pivot": [
            -7.5,
            31,
            0
          ],
          "cubes": [
            {
              "origin": [
                -8.5,
                13,
                -1.5
              ],
              "size": [
                2,
                18,
                3
              ],
              "uv": [
                64,
                28
              ]
            }
          ]
        },
        {
          "name": "left_fingers",
          "parent": "left_forearm",
          "pivot": [
            -7.5,
            13,
            0
          ],
          "cubes": [
            {
              "origin": [
                -8.5,
                6,
                -2
              ],
              "size": [
                2,
                7,
                4
              ],
              "uv": [
                74,
                28
              ]
            }
          ]
        },
        {
          "name": "right_upper_arm",
          "parent": "upper_torso",
          "pivot": [
            6,
            49,
            0
          ],
          "cubes": [
            {
              "origin": [
                6,
                31,
                -1.5
              ],
              "size": [
                3,
                19,
                3
              ],
              "uv": [
                52,
                28
              ],
              "mirror": true
            }
          ]
        },
        {
          "name": "right_forearm",
          "parent": "right_upper_arm",
          "pivot": [
            7.5,
            31,
            0
          ],
          "cubes": [
            {
              "origin": [
                6.5,
                13,
                -1.5
              ],
              "size": [
                2,
                18,
                3
              ],
              "uv": [
                64,
                28
              ],
              "mirror": true
            }
          ]
        },
        {
          "name": "right_fingers",
          "parent": "right_forearm",
          "pivot": [
            7.5,
            13,
            0
          ],
          "cubes": [
            {
              "origin": [
                6.5,
                6,
                -2
              ],
              "size": [
                2,
                7,
                4
              ],
              "uv": [
                74,
                28
              ],
              "mirror": true
            }
          ]
        },
        {
          "name": "left_thigh",
          "parent": "pelvis",
          "pivot": [
            -3,
            24,
            0
          ],
          "cubes": [
            {
              "origin": [
                -4.5,
                12,
                -1.5
              ],
              "size": [
                3,
                12,
                3
              ],
              "uv": [
                0,
                38
              ]
            }
          ]
        },
        {
          "name": "left_shin",
          "parent": "left_thigh",
          "pivot": [
            -3,
            12,
            0
          ],
          "cubes": [
            {
              "origin": [
                -4,
                0,
                -1.5
              ],
              "size": [
                2,
                12,
                3
              ],
              "uv": [
                12,
                38
              ]
            }
          ]
        },
        {
          "name": "right_thigh",
          "parent": "pelvis",
          "pivot": [
            3,
            24,
            0
          ],
          "cubes": [
            {
              "origin": [
                1.5,
                12,
                -1.5
              ],
              "size": [
                3,
                12,
                3
              ],
              "uv": [
                0,
                38
              ],
              "mirror": true
            }
          ]
        },
        {
          "name": "right_shin",
          "parent": "right_thigh",
          "pivot": [
            3,
            12,
            0
          ],
          "cubes": [
            {
              "origin": [
                2,
                0,
                -1.5
              ],
              "size": [
                2,
                12,
                3
              ],
              "uv": [
                12,
                38
              ],
              "mirror": true
            }
          ]
        }
      ]
    }
  ]
}
```

## Dosya: `src/main/resources/assets/dread/geo/grinning_stalker.geo.json`
```json
{
  "format_version": "1.12.0",
  "minecraft:geometry": [
    {
      "description": {
        "identifier": "geometry.dread.grinning_stalker",
        "texture_width": 128,
        "texture_height": 128,
        "visible_bounds_width": 3.0,
        "visible_bounds_height": 4.5,
        "visible_bounds_offset": [
          0,
          1.75,
          0
        ]
      },
      "bones": [
        {
          "name": "root",
          "pivot": [
            0,
            0,
            0
          ]
        },
        {
          "name": "pelvis",
          "parent": "root",
          "pivot": [
            0,
            26,
            0
          ],
          "cubes": [
            {
              "origin": [
                -4.5,
                24,
                -2.5
              ],
              "size": [
                9,
                5,
                5
              ],
              "uv": [
                0,
                28
              ]
            }
          ]
        },
        {
          "name": "lower_torso",
          "parent": "pelvis",
          "pivot": [
            0,
            29,
            0
          ],
          "cubes": [
            {
              "origin": [
                -4,
                29,
                -2
              ],
              "size": [
                8,
                10,
                4
              ],
              "uv": [
                28,
                28
              ]
            }
          ]
        },
        {
          "name": "upper_torso",
          "parent": "lower_torso",
          "pivot": [
            0,
            39,
            0
          ],
          "cubes": [
            {
              "origin": [
                -6,
                39,
                -3
              ],
              "size": [
                12,
                12,
                6
              ],
              "uv": [
                0,
                10
              ]
            }
          ]
        },
        {
          "name": "neck",
          "parent": "upper_torso",
          "pivot": [
            0,
            51,
            0
          ],
          "cubes": [
            {
              "origin": [
                -2,
                50,
                -2
              ],
              "size": [
                4,
                4,
                4
              ],
              "uv": [
                48,
                0
              ]
            }
          ]
        },
        {
          "name": "head",
          "parent": "neck",
          "pivot": [
            0,
            54,
            0
          ],
          "cubes": [
            {
              "origin": [
                -4.5,
                53,
                -4.5
              ],
              "size": [
                9,
                9,
                9
              ],
              "uv": [
                0,
                0
              ]
            }
          ]
        },
        {
          "name": "lower_jaw",
          "parent": "head",
          "pivot": [
            0,
            54,
            -1
          ],
          "cubes": [
            {
              "origin": [
                -4.5,
                51,
                -4.5
              ],
              "size": [
                9,
                3,
                9
              ],
              "uv": [
                36,
                10
              ]
            }
          ]
        },
        {
          "name": "left_upper_arm",
          "parent": "upper_torso",
          "pivot": [
            -6,
            49,
            0
          ],
          "cubes": [
            {
              "origin": [
                -9,
                31,
                -1.5
              ],
              "size": [
                3,
                19,
                3
              ],
              "uv": [
                52,
                28
              ]
            }
          ]
        },
        {
          "name": "left_forearm",
          "parent": "left_upper_arm",
          "pivot": [
            -7.5,
            31,
            0
          ],
          "cubes": [
            {
              "origin": [
                -8.5,
                13,
                -1.5
              ],
              "size": [
                2,
                18,
                3
              ],
              "uv": [
                64,
                28
              ]
            }
          ]
        },
        {
          "name": "left_fingers",
          "parent": "left_forearm",
          "pivot": [
            -7.5,
            13,
            0
          ],
          "cubes": [
            {
              "origin": [
                -8.5,
                6,
                -2
              ],
              "size": [
                2,
                7,
                4
              ],
              "uv": [
                74,
                28
              ]
            }
          ]
        },
        {
          "name": "right_upper_arm",
          "parent": "upper_torso",
          "pivot": [
            6,
            49,
            0
          ],
          "cubes": [
            {
              "origin": [
                6,
                31,
                -1.5
              ],
              "size": [
                3,
                19,
                3
              ],
              "uv": [
                52,
                28
              ],
              "mirror": true
            }
          ]
        },
        {
          "name": "right_forearm",
          "parent": "right_upper_arm",
          "pivot": [
            7.5,
            31,
            0
          ],
          "cubes": [
            {
              "origin": [
                6.5,
                13,
                -1.5
              ],
              "size": [
                2,
                18,
                3
              ],
              "uv": [
                64,
                28
              ],
              "mirror": true
            }
          ]
        },
        {
          "name": "right_fingers",
          "parent": "right_forearm",
          "pivot": [
            7.5,
            13,
            0
          ],
          "cubes": [
            {
              "origin": [
                6.5,
                6,
                -2
              ],
              "size": [
                2,
                7,
                4
              ],
              "uv": [
                74,
                28
              ],
              "mirror": true
            }
          ]
        },
        {
          "name": "left_thigh",
          "parent": "pelvis",
          "pivot": [
            -3,
            24,
            0
          ],
          "cubes": [
            {
              "origin": [
                -4.5,
                12,
                -1.5
              ],
              "size": [
                3,
                12,
                3
              ],
              "uv": [
                0,
                38
              ]
            }
          ]
        },
        {
          "name": "left_shin",
          "parent": "left_thigh",
          "pivot": [
            -3,
            12,
            0
          ],
          "cubes": [
            {
              "origin": [
                -4,
                0,
                -1.5
              ],
              "size": [
                2,
                12,
                3
              ],
              "uv": [
                12,
                38
              ]
            }
          ]
        },
        {
          "name": "right_thigh",
          "parent": "pelvis",
          "pivot": [
            3,
            24,
            0
          ],
          "cubes": [
            {
              "origin": [
                1.5,
                12,
                -1.5
              ],
              "size": [
                3,
                12,
                3
              ],
              "uv": [
                0,
                38
              ],
              "mirror": true
            }
          ]
        },
        {
          "name": "right_shin",
          "parent": "right_thigh",
          "pivot": [
            3,
            12,
            0
          ],
          "cubes": [
            {
              "origin": [
                2,
                0,
                -1.5
              ],
              "size": [
                2,
                12,
                3
              ],
              "uv": [
                12,
                38
              ],
              "mirror": true
            }
          ]
        }
      ]
    }
  ]
}
```

## Dosya: `src/main/resources/assets/dread/geo/herobrine.geo.json`
```json
{
  "format_version": "1.12.0",
  "minecraft:geometry": [
    {
      "description": {
        "identifier": "geometry.dread.herobrine",
        "texture_width": 64,
        "texture_height": 64,
        "visible_bounds_width": 2.0,
        "visible_bounds_height": 3.0,
        "visible_bounds_offset": [
          0,
          1.0,
          0
        ]
      },
      "bones": [
        {
          "name": "root",
          "pivot": [
            0,
            0,
            0
          ]
        },
        {
          "name": "pelvis",
          "parent": "root",
          "pivot": [
            0,
            12,
            0
          ]
        },
        {
          "name": "upper_torso",
          "parent": "pelvis",
          "pivot": [
            0,
            18,
            0
          ],
          "cubes": [
            {
              "origin": [
                -4,
                12,
                -2
              ],
              "size": [
                8,
                12,
                4
              ],
              "uv": [
                16,
                16
              ]
            }
          ]
        },
        {
          "name": "head",
          "parent": "upper_torso",
          "pivot": [
            0,
            24,
            0
          ],
          "cubes": [
            {
              "origin": [
                -4,
                24,
                -4
              ],
              "size": [
                8,
                8,
                8
              ],
              "uv": [
                0,
                0
              ]
            }
          ]
        },
        {
          "name": "lower_jaw",
          "parent": "head",
          "pivot": [
            0,
            24,
            0
          ]
        },
        {
          "name": "left_upper_arm",
          "parent": "upper_torso",
          "pivot": [
            -5,
            22,
            0
          ],
          "cubes": [
            {
              "origin": [
                -8,
                12,
                -2
              ],
              "size": [
                4,
                12,
                4
              ],
              "uv": [
                40,
                16
              ]
            }
          ]
        },
        {
          "name": "right_upper_arm",
          "parent": "upper_torso",
          "pivot": [
            5,
            22,
            0
          ],
          "cubes": [
            {
              "origin": [
                4,
                12,
                -2
              ],
              "size": [
                4,
                12,
                4
              ],
              "uv": [
                40,
                16
              ],
              "mirror": true
            }
          ]
        },
        {
          "name": "left_thigh",
          "parent": "pelvis",
          "pivot": [
            -1.9,
            12,
            0
          ],
          "cubes": [
            {
              "origin": [
                -3.9,
                0,
                -2
              ],
              "size": [
                4,
                12,
                4
              ],
              "uv": [
                0,
                16
              ]
            }
          ]
        },
        {
          "name": "right_thigh",
          "parent": "pelvis",
          "pivot": [
            1.9,
            12,
            0
          ],
          "cubes": [
            {
              "origin": [
                -0.1,
                0,
                -2
              ],
              "size": [
                4,
                12,
                4
              ],
              "uv": [
                0,
                16
              ],
              "mirror": true
            }
          ]
        }
      ]
    }
  ]
}
```

## Dosya: `src/main/resources/assets/dread/geo/pitch_grin.geo.json`
```json
{
  "format_version": "1.12.0",
  "minecraft:geometry": [
    {
      "description": {
        "identifier": "geometry.dread.grinning_stalker",
        "texture_width": 128,
        "texture_height": 128,
        "visible_bounds_width": 3.0,
        "visible_bounds_height": 4.5,
        "visible_bounds_offset": [
          0,
          1.75,
          0
        ]
      },
      "bones": [
        {
          "name": "root",
          "pivot": [
            0,
            0,
            0
          ]
        },
        {
          "name": "pelvis",
          "parent": "root",
          "pivot": [
            0,
            26,
            0
          ],
          "cubes": [
            {
              "origin": [
                -4.5,
                24,
                -2.5
              ],
              "size": [
                9,
                5,
                5
              ],
              "uv": [
                0,
                28
              ]
            }
          ]
        },
        {
          "name": "lower_torso",
          "parent": "pelvis",
          "pivot": [
            0,
            29,
            0
          ],
          "cubes": [
            {
              "origin": [
                -4,
                29,
                -2
              ],
              "size": [
                8,
                10,
                4
              ],
              "uv": [
                28,
                28
              ]
            }
          ]
        },
        {
          "name": "upper_torso",
          "parent": "lower_torso",
          "pivot": [
            0,
            39,
            0
          ],
          "cubes": [
            {
              "origin": [
                -6,
                39,
                -3
              ],
              "size": [
                12,
                12,
                6
              ],
              "uv": [
                0,
                10
              ]
            }
          ]
        },
        {
          "name": "neck",
          "parent": "upper_torso",
          "pivot": [
            0,
            51,
            0
          ],
          "cubes": [
            {
              "origin": [
                -2,
                50,
                -2
              ],
              "size": [
                4,
                4,
                4
              ],
              "uv": [
                48,
                0
              ]
            }
          ]
        },
        {
          "name": "head",
          "parent": "neck",
          "pivot": [
            0,
            54,
            0
          ],
          "cubes": [
            {
              "origin": [
                -4.5,
                53,
                -4.5
              ],
              "size": [
                9,
                9,
                9
              ],
              "uv": [
                0,
                0
              ]
            }
          ]
        },
        {
          "name": "lower_jaw",
          "parent": "head",
          "pivot": [
            0,
            54,
            -1
          ],
          "cubes": [
            {
              "origin": [
                -4.5,
                51,
                -4.5
              ],
              "size": [
                9,
                3,
                9
              ],
              "uv": [
                36,
                10
              ]
            }
          ]
        },
        {
          "name": "left_upper_arm",
          "parent": "upper_torso",
          "pivot": [
            -6,
            49,
            0
          ],
          "cubes": [
            {
              "origin": [
                -9,
                31,
                -1.5
              ],
              "size": [
                3,
                19,
                3
              ],
              "uv": [
                52,
                28
              ]
            }
          ]
        },
        {
          "name": "left_forearm",
          "parent": "left_upper_arm",
          "pivot": [
            -7.5,
            31,
            0
          ],
          "cubes": [
            {
              "origin": [
                -8.5,
                13,
                -1.5
              ],
              "size": [
                2,
                18,
                3
              ],
              "uv": [
                64,
                28
              ]
            }
          ]
        },
        {
          "name": "left_fingers",
          "parent": "left_forearm",
          "pivot": [
            -7.5,
            13,
            0
          ],
          "cubes": [
            {
              "origin": [
                -8.5,
                6,
                -2
              ],
              "size": [
                2,
                7,
                4
              ],
              "uv": [
                74,
                28
              ]
            }
          ]
        },
        {
          "name": "right_upper_arm",
          "parent": "upper_torso",
          "pivot": [
            6,
            49,
            0
          ],
          "cubes": [
            {
              "origin": [
                6,
                31,
                -1.5
              ],
              "size": [
                3,
                19,
                3
              ],
              "uv": [
                52,
                28
              ],
              "mirror": true
            }
          ]
        },
        {
          "name": "right_forearm",
          "parent": "right_upper_arm",
          "pivot": [
            7.5,
            31,
            0
          ],
          "cubes": [
            {
              "origin": [
                6.5,
                13,
                -1.5
              ],
              "size": [
                2,
                18,
                3
              ],
              "uv": [
                64,
                28
              ],
              "mirror": true
            }
          ]
        },
        {
          "name": "right_fingers",
          "parent": "right_forearm",
          "pivot": [
            7.5,
            13,
            0
          ],
          "cubes": [
            {
              "origin": [
                6.5,
                6,
                -2
              ],
              "size": [
                2,
                7,
                4
              ],
              "uv": [
                74,
                28
              ],
              "mirror": true
            }
          ]
        },
        {
          "name": "left_thigh",
          "parent": "pelvis",
          "pivot": [
            -3,
            24,
            0
          ],
          "cubes": [
            {
              "origin": [
                -4.5,
                12,
                -1.5
              ],
              "size": [
                3,
                12,
                3
              ],
              "uv": [
                0,
                38
              ]
            }
          ]
        },
        {
          "name": "left_shin",
          "parent": "left_thigh",
          "pivot": [
            -3,
            12,
            0
          ],
          "cubes": [
            {
              "origin": [
                -4,
                0,
                -1.5
              ],
              "size": [
                2,
                12,
                3
              ],
              "uv": [
                12,
                38
              ]
            }
          ]
        },
        {
          "name": "right_thigh",
          "parent": "pelvis",
          "pivot": [
            3,
            24,
            0
          ],
          "cubes": [
            {
              "origin": [
                1.5,
                12,
                -1.5
              ],
              "size": [
                3,
                12,
                3
              ],
              "uv": [
                0,
                38
              ],
              "mirror": true
            }
          ]
        },
        {
          "name": "right_shin",
          "parent": "right_thigh",
          "pivot": [
            3,
            12,
            0
          ],
          "cubes": [
            {
              "origin": [
                2,
                0,
                -1.5
              ],
              "size": [
                2,
                12,
                3
              ],
              "uv": [
                12,
                38
              ],
              "mirror": true
            }
          ]
        }
      ]
    }
  ]
}
```

## Dosya: `src/main/resources/assets/dread/lang/en_us.json`
```json
{
  "itemGroup.dread.dread_group": "DREAD — Korku & Boyut",
  "block.dread.charred_bone_block": "Yanmış Kemik Bloğu (Charred Bone Block)",
  "block.dread.umbral_portal_block": "Umbral Hollow Portalı",
  "block.dread.ash_layer": "Kül Katmanı",
  "block.dread.ashen_soil": "Küllü Toprak",
  "block.dread.umbral_stone": "Umbral Kaya",
  "block.dread.petrified_log": "Fosilleşmiş Ağaç Gövdesi",
  "block.dread.watching_bark": "İzleyen Kabuk",
  "block.dread.ash_bloom": "Kül Çiçeği",
  "block.dread.umbral_obelisk_block": "Umbral Dikilitaş",
  "block.dread.herobrine_totem_block": "Rünlü Herobrine Totemi",
  "item.dread.charred_bone_shard": "Yanmış Kemik Parçası",
  "item.dread.grinning_heart": "Sırıtan Kalp (Grinning Heart)",
  "item.dread.grinning_heart.desc1": "4x5 Yanmış Kemik Bloğu çerçevesini ateşlemek için kullanılır.",
  "item.dread.grinning_heart.desc2": "Avucunda hala soğuk bir ritimle atıyor...",
  "item.dread.umbral_core": "Apex Dread Çekirdeği",
  "item.dread.lost_journal_1": "Kayıp Günlük — Parça I",
  "item.dread.lost_journal_2": "Kayıp Günlük — Parça II",
  "item.dread.lost_journal_3": "Kayıp Günlük — Parça III",
  "item.dread.lost_journal_4": "Kayıp Günlük — Parça IV",
  "item.dread.lost_journal_5": "Kayıp Günlük — Parça V",
  "item.dread.lost_journal.hint": "Okumak için sağ tıkla",
  "item.dread.tome_of_white_eyes": "Beyaz Gözlerin Kitabı (Totem Rehberi)",
  "item.dread.tome_of_white_eyes.desc1": "Apex Dread'in koruduğu kadim ritüel rehberi.",
  "item.dread.tome_of_white_eyes.desc2": "Herobrine Toteminin kurulumunu açıklar.",
  "item.dread.eclipse_of_the_hollow": "Eclipse of the Hollow (Efsanevi Kılıç)",
  "item.dread.eclipse_of_the_hollow.desc1": "EFSANEVİ SON OYUN SİLAHI",
  "item.dread.eclipse_of_the_hollow.desc2": "Sağ Tık (Umbral Cleave): 12 blokluk gölge yarığı fırlatır ve karanlık verir.",
  "item.dread.eclipse_of_the_hollow.desc3": "Pasif: Dread artışını %75 azaltır.",
  "entity.dread.grinning_stalker": "The Grinning Stalker",
  "entity.dread.pitch_grin": "The Pitch Grin",
  "entity.dread.apex_dread": "APEX DREAD — Çukurun Bekçisi",
  "entity.dread.herobrine_apparition": "Herobrine",
  "entity.dread.herobrine_boss": "HEROBRINE — Korkunun Mimarı",
  "message.dread.totem_overworld_only": "Bu ritüel yalnızca OVERWORLD boyutunda çalışır! Başka boyutta uyanamaz.",
  "message.dread.totem_invalid_structure": "Totem yapısı eksik! Alt: 3x3 Altın Blok, Orta: Totem + 4 Kızıltaş Meşalesi, Üst: Netherrack.",
  "message.dread.cannot_sleep_terrified": "Gözlerini kapattığında o sabit sırıtışı görüyorsun... (Dread 500+ iken uyuyamazsın!)",
  "journal.dread.chapter_1.title": "[GÜNLÜK PARÇA 1/5 — GÖZLEMEVİ KAYDI: UZAKTAKİ SİLÜET]",
  "journal.dread.chapter_1.body": "Karanlıkta geçen her saniye zihnimi kemiriyor (Dread). Işık seviyesi 4'ün altına düştüğünde sisin içinden 3.5 blok boyunda, solgun gri derili ve yüzünde sabit bir sırıtış olan bir varlık (The Watcher) beni izlemeye başlıyor. Yaklaştığımda kayboluyor ama arkamı döndüğümde meşalelerimi tek tek söndürüyor.",
  "journal.dread.chapter_2.title": "[GÜNLÜK PARÇA 2/5 — GÖZLEMEVİ KAYDI: YUVA YIKICI]",
  "journal.dread.chapter_2.body": "Aynı sığınakta çok uzun süre kaldım. Sandıklarımı, fırınımı ve yatağımı kullandığım bölgeyi (HOME BASE) öğrendi. Dün gece dört uzvu üzerinde sürünerek 2 blokluk kapımdan içeri sızdı; önce tüm meşalelerimi ve fenerlerimi kırdı, sonra camlarımı indirdi. Üssümü tamamen karanlığa gömmek istiyor!",
  "journal.dread.chapter_3.title": "[GÜNLÜK PARÇA 3/5 — SIĞINAK KAYDI: DERİNLERDEKİ SIRITIŞ]",
  "journal.dread.chapter_3.body": "Yüzeydeki yaratıktan kaçmak için Y=55'in altına, derin mağaralara indim. Yüzeydeki varlık buraya inemiyor ama korkum (Dread 351+) yükseldiğinde mağara tavanında ışığı tamamen emen simsiyah bir şey (The Pitch Grin) belirdi. Karanlıkta yalnızca bembeyaz sırıtışı görünüyor ve tavandan sessizce sürünüyor.",
  "journal.dread.chapter_4.title": "[GÜNLÜK PARÇA 4/5 — SIĞINAK KAYDI: YANMIŞ KEMİKLER VE KALP]",
  "journal.dread.chapter_4.body": "Bu kabusların kaynağını buldum: The Umbral Hollow. Oraya geçmek için Yanmış Kemik Bloklarından (Charred Bone Blocks) 4 genişlik x 5 yükseklik boyutunda bir çerçeve kurmalı ve merkezini Sırıtan Kalp (Grinning Heart) ile ateşlemeliyim.",
  "journal.dread.chapter_5.title": "[GÜNLÜK PARÇA 5/5 — SON KAYIT: BEYAZ GÖZLERİN SIRRI]",
  "journal.dread.chapter_5.body": "4. Boyuttaki dev gözlü gökyüzünün altında Apex Dread adında devasa bir bekçi yatıyor. Ancak tüm bu yaratıklar asıl efendinin, sislerin içinden bizi izleyen Beyaz Gözlü Varlığın (Herobrine) gölgeleri. Apex Dread'i yenersem, gerçek Totem Rehberini onun sandığından alabilirim.",
  "tome.dread.white_eyes.header": "=== BEYAZ GÖZLERİN KİTABI: HEROBRINE TOTEMİ RİTÜELİ ===",
  "tome.dread.white_eyes.step1": "1. Yalnızca OVERWORLD boyutunda açık bir alan seç (Başka boyutlarda totem çalışmaz!).",
  "tome.dread.white_eyes.step2": "2. Zemine 3x3 Altın Blok (Gold Block) döşe. Tam merkezin üstüne Rünlü Herobrine Totemi koy ve dört yanına 4 Kızıltaş Meşalesi yerleştir.",
  "tome.dread.white_eyes.step3": "3. Totemin tam üstüne 1 Netherrack koy ve Çakmaktaşlı Çelik ile totemi ateşle. Savaş başladığında geri dönüş yoktur!",
  "advancement.dread.root.title": "DREAD: Karanlık Seni İzliyor",
  "advancement.dread.root.desc": "Karanlıkta kalarak ilk kez 100 Dread puanına ulaş.",
  "advancement.dread.survive_the_grin.title": "Yarım Nefes",
  "advancement.dread.survive_the_grin.desc": "Brink of Death veya Hardcore Executioner saldırısından yarım kalple sağ çık.",
  "advancement.dread.enter_umbral_hollow.title": "Kül ve Dev Gözler",
  "advancement.dread.enter_umbral_hollow.desc": "Charred Bone portalını Grinning Heart ile ateşleyip 4. Boyuta (The Umbral Hollow) adım at.",
  "advancement.dread.slay_apex_dread.title": "Sırıtışın Sonu",
  "advancement.dread.slay_apex_dread.desc": "The Umbral Hollow boyutunda Alt Boss Apex Dread'i mağlup et ve Totem Rehberini ele geçir.",
  "advancement.dread.awaken_the_myth.title": "Efsaneyi Uyandır",
  "advancement.dread.awaken_the_myth.desc": "Overworld'de Herobrine Totemini ateşleyerek kadim kabusu çağır.",
  "advancement.dread.conqueror_of_dread.title": "Korkunun Hükümdarı",
  "advancement.dread.conqueror_of_dread.desc": "3 fazlı Herobrine Boss savaşını kazan ve Eclipse of the Hollow kılıcını elde et."
}
```

## Dosya: `src/main/resources/assets/dread/lang/tr_tr.json`
```json
{
  "itemGroup.dread.dread_group": "DREAD — Korku & Boyut",
  "block.dread.charred_bone_block": "Yanmış Kemik Bloğu (Charred Bone Block)",
  "block.dread.umbral_portal_block": "Umbral Hollow Portalı",
  "block.dread.ash_layer": "Kül Katmanı",
  "block.dread.ashen_soil": "Küllü Toprak",
  "block.dread.umbral_stone": "Umbral Kaya",
  "block.dread.petrified_log": "Fosilleşmiş Ağaç Gövdesi",
  "block.dread.watching_bark": "İzleyen Kabuk",
  "block.dread.ash_bloom": "Kül Çiçeği",
  "block.dread.umbral_obelisk_block": "Umbral Dikilitaş",
  "block.dread.herobrine_totem_block": "Rünlü Herobrine Totemi",
  "item.dread.charred_bone_shard": "Yanmış Kemik Parçası",
  "item.dread.grinning_heart": "Sırıtan Kalp (Grinning Heart)",
  "item.dread.grinning_heart.desc1": "4x5 Yanmış Kemik Bloğu çerçevesini ateşlemek için kullanılır.",
  "item.dread.grinning_heart.desc2": "Avucunda hala soğuk bir ritimle atıyor...",
  "item.dread.umbral_core": "Apex Dread Çekirdeği",
  "item.dread.lost_journal_1": "Kayıp Günlük — Parça I",
  "item.dread.lost_journal_2": "Kayıp Günlük — Parça II",
  "item.dread.lost_journal_3": "Kayıp Günlük — Parça III",
  "item.dread.lost_journal_4": "Kayıp Günlük — Parça IV",
  "item.dread.lost_journal_5": "Kayıp Günlük — Parça V",
  "item.dread.lost_journal.hint": "Okumak için sağ tıkla",
  "item.dread.tome_of_white_eyes": "Beyaz Gözlerin Kitabı (Totem Rehberi)",
  "item.dread.tome_of_white_eyes.desc1": "Apex Dread'in koruduğu kadim ritüel rehberi.",
  "item.dread.tome_of_white_eyes.desc2": "Herobrine Toteminin kurulumunu açıklar.",
  "item.dread.eclipse_of_the_hollow": "Eclipse of the Hollow (Efsanevi Kılıç)",
  "item.dread.eclipse_of_the_hollow.desc1": "EFSANEVİ SON OYUN SİLAHI",
  "item.dread.eclipse_of_the_hollow.desc2": "Sağ Tık (Umbral Cleave): 12 blokluk gölge yarığı fırlatır ve karanlık verir.",
  "item.dread.eclipse_of_the_hollow.desc3": "Pasif: Dread artışını %75 azaltır.",
  "entity.dread.grinning_stalker": "The Grinning Stalker",
  "entity.dread.pitch_grin": "The Pitch Grin",
  "entity.dread.apex_dread": "APEX DREAD — Çukurun Bekçisi",
  "entity.dread.herobrine_apparition": "Herobrine",
  "entity.dread.herobrine_boss": "HEROBRINE — Korkunun Mimarı",
  "message.dread.totem_overworld_only": "Bu ritüel yalnızca OVERWORLD boyutunda çalışır! Başka boyutta uyanamaz.",
  "message.dread.totem_invalid_structure": "Totem yapısı eksik! Alt: 3x3 Altın Blok, Orta: Totem + 4 Kızıltaş Meşalesi, Üst: Netherrack.",
  "message.dread.cannot_sleep_terrified": "Gözlerini kapattığında o sabit sırıtışı görüyorsun... (Dread 500+ iken uyuyamazsın!)",
  "journal.dread.chapter_1.title": "[GÜNLÜK PARÇA 1/5 — GÖZLEMEVİ KAYDI: UZAKTAKİ SİLÜET]",
  "journal.dread.chapter_1.body": "Karanlıkta geçen her saniye zihnimi kemiriyor (Dread). Işık seviyesi 4'ün altına düştüğünde sisin içinden 3.5 blok boyunda, solgun gri derili ve yüzünde sabit bir sırıtış olan bir varlık (The Watcher) beni izlemeye başlıyor. Yaklaştığımda kayboluyor ama arkamı döndüğümde meşalelerimi tek tek söndürüyor.",
  "journal.dread.chapter_2.title": "[GÜNLÜK PARÇA 2/5 — GÖZLEMEVİ KAYDI: YUVA YIKICI]",
  "journal.dread.chapter_2.body": "Aynı sığınakta çok uzun süre kaldım. Sandıklarımı, fırınımı ve yatağımı kullandığım bölgeyi (HOME BASE) öğrendi. Dün gece dört uzvu üzerinde sürünerek 2 blokluk kapımdan içeri sızdı; önce tüm meşalelerimi ve fenerlerimi kırdı, sonra camlarımı indirdi. Üssümü tamamen karanlığa gömmek istiyor!",
  "journal.dread.chapter_3.title": "[GÜNLÜK PARÇA 3/5 — SIĞINAK KAYDI: DERİNLERDEKİ SIRITIŞ]",
  "journal.dread.chapter_3.body": "Yüzeydeki yaratıktan kaçmak için Y=55'in altına, derin mağaralara indim. Yüzeydeki varlık buraya inemiyor ama korkum (Dread 351+) yükseldiğinde mağara tavanında ışığı tamamen emen simsiyah bir şey (The Pitch Grin) belirdi. Karanlıkta yalnızca bembeyaz sırıtışı görünüyor ve tavandan sessizce sürünüyor.",
  "journal.dread.chapter_4.title": "[GÜNLÜK PARÇA 4/5 — SIĞINAK KAYDI: YANMIŞ KEMİKLER VE KALP]",
  "journal.dread.chapter_4.body": "Bu kabusların kaynağını buldum: The Umbral Hollow. Oraya geçmek için Yanmış Kemik Bloklarından (Charred Bone Blocks) 4 genişlik x 5 yükseklik boyutunda bir çerçeve kurmalı ve merkezini Sırıtan Kalp (Grinning Heart) ile ateşlemeliyim.",
  "journal.dread.chapter_5.title": "[GÜNLÜK PARÇA 5/5 — SON KAYIT: BEYAZ GÖZLERİN SIRRI]",
  "journal.dread.chapter_5.body": "4. Boyuttaki dev gözlü gökyüzünün altında Apex Dread adında devasa bir bekçi yatıyor. Ancak tüm bu yaratıklar asıl efendinin, sislerin içinden bizi izleyen Beyaz Gözlü Varlığın (Herobrine) gölgeleri. Apex Dread'i yenersem, gerçek Totem Rehberini onun sandığından alabilirim.",
  "tome.dread.white_eyes.header": "=== BEYAZ GÖZLERİN KİTABI: HEROBRINE TOTEMİ RİTÜELİ ===",
  "tome.dread.white_eyes.step1": "1. Yalnızca OVERWORLD boyutunda açık bir alan seç (Başka boyutlarda totem çalışmaz!).",
  "tome.dread.white_eyes.step2": "2. Zemine 3x3 Altın Blok (Gold Block) döşe. Tam merkezin üstüne Rünlü Herobrine Totemi koy ve dört yanına 4 Kızıltaş Meşalesi yerleştir.",
  "tome.dread.white_eyes.step3": "3. Totemin tam üstüne 1 Netherrack koy ve Çakmaktaşlı Çelik ile totemi ateşle. Savaş başladığında geri dönüş yoktur!",
  "advancement.dread.root.title": "DREAD: Karanlık Seni İzliyor",
  "advancement.dread.root.desc": "Karanlıkta kalarak ilk kez 100 Dread puanına ulaş.",
  "advancement.dread.survive_the_grin.title": "Yarım Nefes",
  "advancement.dread.survive_the_grin.desc": "Brink of Death veya Hardcore Executioner saldırısından yarım kalple sağ çık.",
  "advancement.dread.enter_umbral_hollow.title": "Kül ve Dev Gözler",
  "advancement.dread.enter_umbral_hollow.desc": "Charred Bone portalını Grinning Heart ile ateşleyip 4. Boyuta (The Umbral Hollow) adım at.",
  "advancement.dread.slay_apex_dread.title": "Sırıtışın Sonu",
  "advancement.dread.slay_apex_dread.desc": "The Umbral Hollow boyutunda Alt Boss Apex Dread'i mağlup et ve Totem Rehberini ele geçir.",
  "advancement.dread.awaken_the_myth.title": "Efsaneyi Uyandır",
  "advancement.dread.awaken_the_myth.desc": "Overworld'de Herobrine Totemini ateşleyerek kadim kabusu çağır.",
  "advancement.dread.conqueror_of_dread.title": "Korkunun Hükümdarı",
  "advancement.dread.conqueror_of_dread.desc": "3 fazlı Herobrine Boss savaşını kazan ve Eclipse of the Hollow kılıcını elde et."
}
```

## Dosya: `src/main/resources/assets/dread/sounds.json`
```json
{
  "ambient.dread_drone": {
    "sounds": [
      {
        "name": "dread:ambient/dread_drone",
        "stream": true
      }
    ]
  },
  "ambient.umbral_wind": {
    "sounds": [
      {
        "name": "dread:ambient/umbral_wind",
        "stream": true
      }
    ]
  },
  "ambient.fake_footstep": {
    "sounds": [
      "dread:ambient/fake_footstep_1",
      "dread:ambient/fake_footstep_2"
    ]
  },
  "ambient.fake_torch_hiss": {
    "sounds": [
      "dread:ambient/fake_torch_hiss"
    ]
  },
  "ambient.heartbeat": {
    "sounds": [
      "dread:ambient/heartbeat"
    ]
  },
  "entity.stalker.footstep_heavy": {
    "sounds": [
      "dread:entity/stalker/step_1",
      "dread:entity/stalker/step_2"
    ]
  },
  "entity.stalker.bone_click": {
    "sounds": [
      "dread:entity/stalker/bone_click_1",
      "dread:entity/stalker/bone_click_2"
    ]
  },
  "entity.stalker.breathing": {
    "sounds": [
      "dread:entity/stalker/breathing"
    ]
  },
  "entity.stalker.slash": {
    "sounds": [
      "dread:entity/stalker/slash"
    ]
  },
  "entity.stalker.vanish": {
    "sounds": [
      "dread:entity/stalker/vanish"
    ]
  },
  "event.jumpscare.stalker": {
    "sounds": [
      "dread:event/jumpscare_stalker"
    ]
  },
  "entity.pitch_grin.crawl": {
    "sounds": [
      "dread:entity/pitch_grin/crawl_1",
      "dread:entity/pitch_grin/crawl_2"
    ]
  },
  "entity.pitch_grin.shriek_bite": {
    "sounds": [
      "dread:entity/pitch_grin/shriek_bite"
    ]
  },
  "entity.apex_dread.roar": {
    "sounds": [
      "dread:entity/apex_dread/roar"
    ]
  },
  "entity.apex_dread.gaze_charge": {
    "sounds": [
      "dread:entity/apex_dread/gaze_charge"
    ]
  },
  "music.boss.apex_dread": {
    "sounds": [
      {
        "name": "dread:music/boss_apex_dread",
        "stream": true
      }
    ]
  },
  "event.jumpscare.herobrine_static": {
    "sounds": [
      "dread:event/jumpscare_herobrine_static"
    ]
  },
  "event.herobrine.totem_awaken": {
    "sounds": [
      "dread:event/herobrine_totem_awaken"
    ]
  },
  "music.boss.herobrine_phase1": {
    "sounds": [
      {
        "name": "dread:music/boss_herobrine_p1",
        "stream": true
      }
    ]
  },
  "music.boss.herobrine_phase2": {
    "sounds": [
      {
        "name": "dread:music/boss_herobrine_p2",
        "stream": true
      }
    ]
  },
  "music.boss.herobrine_phase3": {
    "sounds": [
      {
        "name": "dread:music/boss_herobrine_p3",
        "stream": true
      }
    ]
  }
}
```

## Dosya: `src/main/resources/data/dread/advancement/awaken_the_myth.json`
```json
{
  "display": {
    "icon": {
      "id": "dread:herobrine_totem_block"
    },
    "title": {
      "translate": "advancement.dread.awaken_the_myth.title"
    },
    "description": {
      "translate": "advancement.dread.awaken_the_myth.desc"
    },
    "frame": "goal",
    "show_toast": true,
    "announce_to_chat": true,
    "hidden": false
  },
  "criteria": {
    "requirement": {
      "trigger": "dread:awaken_herobrine_totem"
    }
  },
  "parent": "dread:slay_apex_dread"
}
```

## Dosya: `src/main/resources/data/dread/advancement/conqueror_of_dread.json`
```json
{
  "display": {
    "icon": {
      "id": "dread:eclipse_of_the_hollow"
    },
    "title": {
      "translate": "advancement.dread.conqueror_of_dread.title"
    },
    "description": {
      "translate": "advancement.dread.conqueror_of_dread.desc"
    },
    "frame": "challenge",
    "show_toast": true,
    "announce_to_chat": true,
    "hidden": false
  },
  "criteria": {
    "requirement": {
      "trigger": "dread:slay_herobrine_boss"
    }
  },
  "parent": "dread:awaken_the_myth"
}
```

## Dosya: `src/main/resources/data/dread/advancement/enter_umbral_hollow.json`
```json
{
  "display": {
    "icon": {
      "id": "dread:charred_bone_block"
    },
    "title": {
      "translate": "advancement.dread.enter_umbral_hollow.title"
    },
    "description": {
      "translate": "advancement.dread.enter_umbral_hollow.desc"
    },
    "frame": "goal",
    "show_toast": true,
    "announce_to_chat": true,
    "hidden": false
  },
  "criteria": {
    "requirement": {
      "trigger": "dread:enter_umbral_hollow"
    }
  },
  "parent": "dread:survive_the_grin"
}
```

## Dosya: `src/main/resources/data/dread/advancement/root.json`
```json
{
  "display": {
    "icon": {
      "id": "dread:lost_journal_1"
    },
    "title": {
      "translate": "advancement.dread.root.title"
    },
    "description": {
      "translate": "advancement.dread.root.desc"
    },
    "frame": "task",
    "show_toast": true,
    "announce_to_chat": true,
    "hidden": false,
    "background": "minecraft:textures/gui/advancements/backgrounds/adventure.png"
  },
  "criteria": {
    "requirement": {
      "trigger": "dread:first_dread_100"
    }
  }
}
```

## Dosya: `src/main/resources/data/dread/advancement/slay_apex_dread.json`
```json
{
  "display": {
    "icon": {
      "id": "dread:umbral_core"
    },
    "title": {
      "translate": "advancement.dread.slay_apex_dread.title"
    },
    "description": {
      "translate": "advancement.dread.slay_apex_dread.desc"
    },
    "frame": "challenge",
    "show_toast": true,
    "announce_to_chat": true,
    "hidden": false
  },
  "criteria": {
    "requirement": {
      "trigger": "dread:slay_apex_dread"
    }
  },
  "parent": "dread:enter_umbral_hollow"
}
```

## Dosya: `src/main/resources/data/dread/advancement/survive_the_grin.json`
```json
{
  "display": {
    "icon": {
      "id": "dread:grinning_heart"
    },
    "title": {
      "translate": "advancement.dread.survive_the_grin.title"
    },
    "description": {
      "translate": "advancement.dread.survive_the_grin.desc"
    },
    "frame": "goal",
    "show_toast": true,
    "announce_to_chat": true,
    "hidden": false
  },
  "criteria": {
    "requirement": {
      "trigger": "dread:survive_half_heart"
    }
  },
  "parent": "dread:root"
}
```

## Dosya: `src/main/resources/data/dread/dimension/umbral_hollow.json`
```json
{
  "type": "dread:umbral_hollow_type",
  "generator": {
    "type": "minecraft:noise",
    "settings": "minecraft:overworld",
    "biome_source": {
      "type": "minecraft:multi_noise",
      "biomes": [
        {
          "biome": "dread:ashen_wastes",
          "parameters": {
            "temperature": 0.0,
            "humidity": -0.5,
            "continentalness": 0.0,
            "erosion": 0.0,
            "weirdness": 0.0,
            "depth": 0.0,
            "offset": 0.0
          }
        },
        {
          "biome": "dread:petrified_grove",
          "parameters": {
            "temperature": -0.2,
            "humidity": 0.4,
            "continentalness": 0.2,
            "erosion": 0.1,
            "weirdness": 0.1,
            "depth": 0.0,
            "offset": 0.0
          }
        },
        {
          "biome": "dread:the_hollow_maw",
          "parameters": {
            "temperature": 0.2,
            "humidity": 0.0,
            "continentalness": -0.3,
            "erosion": -0.2,
            "weirdness": 0.0,
            "depth": 0.0,
            "offset": 0.0
          }
        }
      ]
    }
  }
}
```

## Dosya: `src/main/resources/data/dread/dimension_type/umbral_hollow_type.json`
```json
{
  "ultrawarm": false,
  "natural": false,
  "piglin_safe": false,
  "respawn_anchor_works": true,
  "bed_works": false,
  "has_raids": false,
  "has_skylight": false,
  "has_ceiling": false,
  "coordinate_scale": 1.0,
  "ambient_light": 0.04,
  "fixed_time": 18000,
  "logical_height": 256,
  "effects": "dread:umbral_hollow",
  "infiniburn": "#minecraft:infiniburn_overworld",
  "min_y": 0,
  "height": 256,
  "monster_spawn_light_level": {
    "type": "minecraft:uniform",
    "min_inclusive": 0,
    "max_inclusive": 7
  },
  "monster_spawn_block_light_limit": 0
}
```

## Dosya: `src/main/resources/data/dread/recipe/charred_bone_block.json`
```json
{
  "type": "minecraft:crafting_shaped",
  "pattern": [
    "BSB",
    "SCS",
    "BSB"
  ],
  "key": {
    "B": {
      "item": "minecraft:bone_block"
    },
    "S": {
      "item": "minecraft:soul_sand"
    },
    "C": {
      "item": "minecraft:coal_block"
    }
  },
  "result": {
    "id": "dread:charred_bone_block",
    "count": 4
  }
}
```

## Dosya: `src/main/resources/data/dread/recipe/herobrine_totem_block.json`
```json
{
  "type": "minecraft:crafting_shaped",
  "pattern": [
    "GNG",
    "NUN",
    "GNG"
  ],
  "key": {
    "G": {
      "item": "minecraft:gold_block"
    },
    "N": {
      "item": "minecraft:netherrack"
    },
    "U": {
      "item": "dread:umbral_core"
    }
  },
  "result": {
    "id": "dread:herobrine_totem_block",
    "count": 1
  }
}
```

## Dosya: `src/main/resources/data/dread/worldgen/biome/ashen_wastes.json`
```json
{
  "has_precipitation": false,
  "temperature": 0.2,
  "downfall": 0.0,
  "effects": {
    "fog_color": 855570,
    "sky_color": 526348,
    "water_color": 1315865,
    "water_fog_color": 328965,
    "grass_color_override": 2631720,
    "foliage_color_override": 2631720,
    "particle": {
      "options": {
        "type": "minecraft:ash"
      },
      "probability": 0.045
    }
  },
  "spawners": {
    "monster": [
      {
        "type": "dread:apex_dread",
        "weight": 2,
        "minCount": 1,
        "maxCount": 1
      }
    ],
    "creature": [],
    "ambient": [],
    "water_creature": [],
    "underground_water_creature": [],
    "water_ambient": [],
    "misc": []
  },
  "spawn_costs": {},
  "carvers": {},
  "features": []
}
```

## Dosya: `src/main/resources/data/dread/worldgen/biome/petrified_grove.json`
```json
{
  "has_precipitation": false,
  "temperature": 0.2,
  "downfall": 0.0,
  "effects": {
    "fog_color": 855570,
    "sky_color": 526348,
    "water_color": 1315865,
    "water_fog_color": 328965,
    "grass_color_override": 2631720,
    "foliage_color_override": 2631720,
    "particle": {
      "options": {
        "type": "minecraft:ash"
      },
      "probability": 0.045
    }
  },
  "spawners": {
    "monster": [
      {
        "type": "dread:apex_dread",
        "weight": 2,
        "minCount": 1,
        "maxCount": 1
      }
    ],
    "creature": [],
    "ambient": [],
    "water_creature": [],
    "underground_water_creature": [],
    "water_ambient": [],
    "misc": []
  },
  "spawn_costs": {},
  "carvers": {},
  "features": []
}
```

## Dosya: `src/main/resources/data/dread/worldgen/biome/the_hollow_maw.json`
```json
{
  "has_precipitation": false,
  "temperature": 0.2,
  "downfall": 0.0,
  "effects": {
    "fog_color": 855570,
    "sky_color": 526348,
    "water_color": 1315865,
    "water_fog_color": 328965,
    "grass_color_override": 2631720,
    "foliage_color_override": 2631720,
    "particle": {
      "options": {
        "type": "minecraft:ash"
      },
      "probability": 0.045
    }
  },
  "spawners": {
    "monster": [
      {
        "type": "dread:apex_dread",
        "weight": 2,
        "minCount": 1,
        "maxCount": 1
      }
    ],
    "creature": [],
    "ambient": [],
    "water_creature": [],
    "underground_water_creature": [],
    "water_ambient": [],
    "misc": []
  },
  "spawn_costs": {},
  "carvers": {},
  "features": []
}
```

## Dosya: `src/main/resources/dread.mixins.json`
```json
{
  "required": true,
  "minVersion": "0.8",
  "package": "com.studio.dread.mixin",
  "compatibilityLevel": "JAVA_21",
  "mixins": [
    "LivingEntityDamageMixin",
    "ServerPlayerEntityMixin"
  ],
  "client": [
    "BackgroundRendererMixin"
  ],
  "injectors": {
    "defaultRequire": 1
  }
}
```

## Dosya: `src/main/resources/fabric.mod.json`
```json
{
  "schemaVersion": 1,
  "id": "dread",
  "version": "${version}",
  "name": "DREAD — Psychological Horror & The Umbral Hollow",
  "description": "Profesyonel ölçekli korku modu: Dinamik Dread sistemi, The Grinning Stalker (6 Versiyon), The Pitch Grin, 4. Boyut (The Umbral Hollow), Apex Dread ve 3 Fazlı Herobrine Boss Savaşı.",
  "authors": [
    "Dread Studio Architecture"
  ],
  "license": "All-Rights-Reserved",
  "environment": "*",
  "entrypoints": {
    "main": [
      "com.studio.dread.DreadMod"
    ],
    "client": [
      "com.studio.dread.DreadClient"
    ]
  },
  "mixins": [
    "dread.mixins.json"
  ],
  "depends": {
    "fabricloader": ">=0.16.0",
    "minecraft": "~1.21.1",
    "java": ">=21",
    "fabric-api": "*",
    "geckolib": ">=4.5.8"
  }
}
```
