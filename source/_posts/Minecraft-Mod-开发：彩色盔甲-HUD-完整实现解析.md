---
title: Minecraft Mod 开发：彩色盔甲 HUD 完整实现
date: 2026-10-25 17:00:50
tags:
  - Minecraft
  - 1.21.1
  - NeoForge
  - MOD开发
categories:
  - Minecraft MOD 开发
cover: 'https://img.liuranblog.top/file/1792919777687_post_4.png'
---
> 本文基于一份纯客户端 Minecraft（1.21.1，NeoForge 写法）模组源码，逐文件拆解「彩色盔甲状态栏 HUD」的完整实现。
> 参考于[Colored Armor HUD](https://www.curseforge.com/minecraft/mc-mods/colored-armor-hud) 作者：Lunar201
> 仅分析和供想要完成类似情况的开发者学习
> 该模组**只在客户端工作**，不修改数据包、不向服务端发送任何状态，因此可以在任意服务器上独立生效。

## 功能一览

1. 根据盔甲**材质**动态生成状态栏盾牌图标，不同材质呈现不同颜色/纹理；
2. 混穿盔甲时支持**半格分割渲染**：同一个盾牌图标左半、右半分别显示两件不同盔甲；
3. 完整支持 1.20+ **盔甲纹饰（Trim）**，纹饰颜色正确叠加到 HUD 图标；
4. 皮革盔甲支持**染料染色**，颜色以乘法方式叠加上去；
5. 附魔盔甲附带**流动光带**与**闪烁星光粒子**两种特效；
6. JSON 配置驱动：材质颜色、物品强制色、发光亮度/饱和度/颜色、特效开关全部可配。

---

## 一、整体架构总览

模块由 4 个类组成，职责自上而下分层：

| 类名 | 分层 | 核心职责 |
| --- | --- | --- |
| `ArmorHudColors` | 数据提取层 | 从 `ItemStack` 读取材质 ID、纹饰调色板、皮革染色 RGB，计算最终颜色 |
| `ArmorHudConfig` | 配置管理层 | JSON 读写、十六进制颜色解析、颜色缓存、默认值补全、饱和度工具 |
| `MaterialTextures` | 动态纹理层 | 基于原版盾牌 mask，运行时合成 `DynamicTexture`，处理纹饰/染色，带缓存 |
| `ArmorHudRenderer` | 渲染入口层 | 接管 HUD 盔甲栏，遍历装备槽、分配护甲点数、调用纹理、画分割/流光/星光 |

### 数据流链路

```text
原版 HUD 盔甲栏绘制钩子
        │
        ▼
ArmorHudRenderer.renderColored()
        │  读取每个装备槽
        ├─► ArmorHudColors.materialId() / colorFor() / trimPalette() / leatherTint()
        │        └─► ArmorHudConfig 查询颜色（itemColors > materialColors > defaultColor）
        ├─► MaterialTextures.iconFor()  合成/缓存动态纹理
        └─► 绘制盾牌 + 附魔流光 + 星光，返回 true 拦截原版绘制
```

---

## 二、`ArmorHudColors`：盔甲信息提取层

> 脱敏包名：`package yourmod.integration.coloredarmor;`

这个类**不做任何渲染**，只负责把一件盔甲 `ItemStack` 里渲染需要的元数据全部提取出来，让上层只面向「材质 ID / 颜色 / 纹饰 / 染色」这些干净的数据。

### 完整脱敏源码

```java
package yourmod.integration.coloredarmor;

import java.util.Optional;
import net.minecraft.core.component.DataComponents;
import net.minecraft.core.registries.BuiltInRegistries;
import net.minecraft.resources.ResourceLocation;
import net.minecraft.world.item.ArmorItem;
import net.minecraft.world.item.ItemStack;
import net.minecraft.world.item.armortrim.ArmorTrim;
import net.minecraft.world.item.armortrim.TrimMaterial;
import net.minecraft.world.item.component.DyedItemColor;

public final class ArmorHudColors {
    private ArmorHudColors() {
    }

    // 颜色解析：物品ID强制色 > 材质色 > 全局默认色
    public static int colorFor(ItemStack stack, ArmorHudConfig cfg) {
        ResourceLocation itemId = BuiltInRegistries.ITEM.getKey(stack.getItem());
        if (itemId != null) {
            Integer override = cfg.itemColor(itemId.toString());
            if (override != null) {
                return override;
            }
        }

        if (stack.getItem() instanceof ArmorItem armor) {
            Optional<ResourceLocation> matKey = armor.getMaterial().unwrapKey().map((key) -> key.location());
            if (matKey.isPresent()) {
                Integer matColor = cfg.materialColor(matKey.get().toString());
                if (matColor != null) {
                    return matColor;
                }
            }
        }

        return cfg.defaultColorRGB();
    }

    // 提取盔甲材质的命名空间ID，例如 minecraft:netherite
    public static String materialId(ItemStack stack) {
        if (stack.getItem() instanceof ArmorItem armor) {
            Optional<ResourceLocation> matKey = armor.getMaterial().unwrapKey().map((key) -> key.location());
            if (matKey.isPresent()) {
                return matKey.get().toString();
            }
        }
        return null;
    }

    // 读取盔甲纹饰调色板名，并处理材质对特定盔甲的调色板覆写
    public static String trimPalette(ItemStack stack) {
        ArmorTrim trim = stack.get(DataComponents.TRIM);
        if (trim == null) {
            return null;
        } else {
            TrimMaterial material = trim.material().value();
            String name = material.assetName();
            if (stack.getItem() instanceof ArmorItem armor) {
                String override = material.overrideArmorMaterials().get(armor.getMaterial());
                if (override != null) {
                    name = override;
                }
            }
            return name;
        }
    }

    // 仅皮革盔甲返回染色RGB；其余返回 -1 作为“无染色”标记
    public static int leatherTint(ItemStack stack) {
        return "minecraft:leather".equals(materialId(stack))
                ? DyedItemColor.getOrDefault(stack, -6265536) & 16777215
                : -1;
    }
}
```

### 逐方法分析

#### 1. `colorFor(stack, cfg)` —— 三级颜色回退

颜色来源按优先级依次查找：

1. **`itemColors`（物品 ID 强制覆盖）**：用 `BuiltInRegistries.ITEM.getKey()` 拿到物品的注册 ID（如 `minecraft:diamond_chestplate`），能覆盖同材质不同物品的颜色，适合给模组里某一件特殊盔甲单独配色；
2. **`materialColors`（盔甲材质色）**：通过 `armor.getMaterial().unwrapKey().map(Key::location)` 拿到材质 ID（如 `minecraft:diamond`），按材质批量配色；
3. **`defaultColorRGB()`（全局默认色）**：前面都没命中时兜底。

> 设计意图：物品级覆盖材质级，材质级覆盖默认级，形成「细粒度 → 粗粒度」的回退链，玩家既可以整材质统一配色，也可以对单件物品破例。

#### 2. `materialId(stack)` —— 材质 ID 提取

把盔甲材质的 `ResourceLocation` 转成字符串（`minecraft:iron`、`minecraft:netherite` …）。这个字符串同时充当：配置文件的 key、动态纹理缓存的 key、纹理路径 switch 的判断条件。非盔甲返回 `null`。

#### 3. `trimPalette(stack)` —— 纹饰调色板（关键兼容点）

从数据组件 `DataComponents.TRIM` 读出 `ArmorTrim`，再取 `TrimMaterial.assetName()` 作为调色板名。

容易被同类 Mod 漏掉的细节是 `material.overrideArmorMaterials()`：原版允许某个纹饰材质**针对特定盔甲材质覆写一套不同的调色板**（例如同一纹饰在铁和钻石上呈现不同颜色）。代码显式查了这张覆写表，因此 HUD 上的纹饰颜色与游戏内盔甲实体渲染保持一致。

#### 4. `leatherTint(stack)` —— 皮革染色

- 只有材质等于 `minecraft:leather` 才继续；
- `DyedItemColor.getOrDefault(stack, -6265536)` 取染色，`-6265536`（0xFFA06540 附近）是原版皮革默认棕；
- `& 16777215`（0xFFFFFF）清掉高位 alpha，只保留 24 位 RGB；
- 非皮革返回 `-1`，上层用 `tint >= 0` 判断是否执行染色乘法。

---

## 三、`ArmorHudConfig`：JSON 配置与颜色工具

> 脱敏包名：`package yourmod.integration.coloredarmor;`
> 脱敏配置路径：`config/yourmod_armor_hud.json`

负责配置文件的加载/保存、十六进制颜色解析、解析结果缓存、默认值补全，以及全模块复用的饱和度算法。

### 3.1 字段表

| 字段 | 类型 | 含义 |
| --- | --- | --- |
| `materialColors` | `Map<String,String>` | 材质 ID → 十六进制颜色，如 `minecraft:diamond → #5BE7E0` |
| `itemColors` | `Map<String,String>` | 物品 ID → 颜色，优先级高于材质 |
| `defaultColor` | `String` | 全局兜底色，默认 `#A8A8A8` |
| `stars` | `boolean` | 星光/流光特效总开关 |
| `trimSaturation` | `Double` | 纹饰饱和度倍率 |
| `glowBrightness` | `Double` | 附魔流光峰值亮度（0~1），默认 0.42 |
| `glowSaturation` | `Double` | 流光颜色饱和度倍率 |
| `glowColor` | `String` | 流光基色，默认 `#FFEEB3` |
| `armorBar` | `Boolean` | 盔甲栏总开关 |
| `warnLowDurability` | `Boolean` | 低耐久警告开关 |
| `colorArmorBar` | `Boolean` | 是否启用彩色盔甲栏（关闭则回退原版） |
| `durabilityWarnThreshold` | `Double` | 低耐久阈值，默认 0.05 |
| `hudVisible` | `transient boolean` | 运行期可见开关，**不写进 JSON** |
| `matCache` / `itemCache` | `transient Map` | 颜色解析结果缓存，不序列化 |
| `defaultRGB` | `transient int` | 默认色解析后的整数，不序列化 |

### 3.2 颜色解析 `parseHex`

```java
static Integer parseHex(String s) {
    if (s == null) {
        return null;
    }
    String t = s.trim();
    if (t.startsWith("#")) {
        t = t.substring(1);
    } else if (t.startsWith("0x") || t.startsWith("0X")) {
        t = t.substring(2);
    }
    try {
        return (int) (Long.parseLong(t, 16) & 16777215L);
    } catch (NumberFormatException e) {
        return null;
    }
}
```

- 同时兼容 `#RRGGBB` 与 `0xRRGGBB` 两种写法；
- 用 `Long.parseLong(..., 16)` 再 `& 0xFFFFFF`，避免 6 位以上/带 alpha 的输入抛错或带入高位；
- 格式错误返回 `null`，由调用方走兜底色，而不是崩溃。

### 3.3 饱和度算法 `applySaturation`（核心工具）

```java
public static int applySaturation(int rgb, double sat) {
    int r = rgb >> 16 & 0xFF;
    int g = rgb >> 8 & 0xFF;
    int b = rgb & 0xFF;
    double luma = 0.299 * r + 0.587 * g + 0.114 * b;
    int nr = clampByte((int) Math.round(luma + (r - luma) * sat));
    int ng = clampByte((int) Math.round(luma + (g - luma) * sat));
    int nb = clampByte((int) Math.round(luma + (b - luma) * sat));
    return nr << 16 | ng << 8 | nb;
}
```

算法本质：**先算亮度 luma，再围绕 luma 缩放「颜色与灰度的差值」**。

```text
新通道 = luma + (原通道 - luma) × sat
```

- `sat = 1.0`：原色不变；
- `sat = 0`：变成纯灰度（只保留 luma）；
- `sat > 1`：颜色更艳。

关键好处是**保持亮度不变**：直接对 RGB 乘系数会同时改变明暗观感，而本算法只动「色彩鲜艳程度」，纹饰和流光调色后不会整体变亮/变暗。`clampByte` 把结果夹在 0~255。

### 3.4 带缓存的 `resolve`

```java
private static Integer resolve(Map<String, String> src, Map<String, Integer> cache, String id) {
    if (cache.containsKey(id)) {
        return cache.get(id);
    }
    Integer value = src == null ? null : parseHex(src.get(id));
    cache.put(id, value);
    return value;
}
```

渲染每帧都在查询颜色，十六进制字符串解析若每帧重复做是浪费。`resolve` 把「解析成功」和「解析失败（null）」都缓存下来，一件盔甲一帧内只解析一次。

### 3.5 加载容错与默认值补全

`load()` 的防御逻辑层层兜底：

1. 配置文件不存在 → 生成默认配置并落盘；
2. JSON 解析抛任何异常 → 直接回退 `defaults()`，**游戏不会因为配置写坏而崩溃**；
3. 解析成功后，对每个可空字段逐一判空补默认（`trimSaturation`、`glowBrightness`、`glowColor` …）；
4. `materialColors` 用 `putIfAbsent` 把内置默认材质色补进缺失项，**不覆盖玩家已经改过的值**；
5. 最后重建缓存、解析默认色。

### 3.6 保存联动清缓存

```java
public static void save() {
    if (INSTANCE != null) {
        Integer dc = parseHex(INSTANCE.defaultColor);
        INSTANCE.defaultRGB = dc != null ? dc : 11053224;
        save(INSTANCE, configPath());
        MaterialTextures.clearCache();
    }
}
```

保存配置后立即调用 `MaterialTextures.clearCache()`，让旧的动态纹理作废、下次重新合成，避免「改了颜色图标却不变」。

---

## 四、`MaterialTextures`：运行时动态纹理生成（技术核心）

> 脱敏包名：`package yourmod.integration.coloredarmor;`

原版盔甲 HUD 使用固定 sprite `hud/armor_full`。本模组**不在资源包里堆放大量盔甲 PNG**，而是在游戏运行时：读取盔甲物品纹理 → 套原版盾牌 mask → 用 `NativeImage` 逐像素合成 → 包成 `DynamicTexture` 注册给 GPU。这是整个模块最复杂、也最值得讲的部分。

### 4.1 常量与缓存

```java
private static final ResourceLocation MASK =
        ResourceLocation.withDefaultNamespace("textures/gui/sprites/hud/armor_full.png");
private static final Map<String, ResourceLocation> CACHE = new HashMap<>();
private static final ResourceLocation MISSING =
        ResourceLocation.withDefaultNamespace("coloredarmorhud_missing");
```

- `MASK`：原版盾牌形状蒙版，提供轮廓与明暗；
- `CACHE`：合成结果缓存，key 形如 `mat:minecraft:diamond|trim:diamond|tint:-1` 或 `col:11053224`；
- `MISSING`：**负缓存标记**——某个材质合成失败后，把 `MISSING` 存进缓存，避免每帧反复尝试读一张不存在的贴图。

`cached()` 包装器同时处理成功与失败：

```java
private static ResourceLocation cached(String key, Builder builder) {
    ResourceLocation cachedLoc = CACHE.get(key);
    if (cachedLoc != null) {
        return cachedLoc == MISSING ? null : cachedLoc;
    }
    ResourceLocation built;
    try {
        built = builder.build();
    } catch (Exception e) {
        built = null;
    }
    CACHE.put(key, built == null ? MISSING : built);
    return built;
}
```

### 4.2 两个入口

```java
public static ResourceLocation iconFor(String materialId, int fallbackColor, String trimPalette, int tint) {
    if (materialId != null) {
        String texPath = texturePath(materialId);
        if (texPath != null) {
            String key = "mat:" + materialId + "|trim:" + (trimPalette == null ? "" : trimPalette) + "|tint:" + tint;
            ResourceLocation loc = cached(key, () -> buildFromTexture(texPath, fallbackColor, trimPalette, tint));
            if (loc != null) {
                return loc;
            }
        }
    }
    return forColor(fallbackColor);
}

public static ResourceLocation forColor(int rgb) {
    ResourceLocation loc = cached("col:" + (rgb & 16777215), () -> buildFromColor(rgb));
    return loc != null ? loc : MASK;
}
```

- `iconFor`：优先用材质物品纹理合成；材质无映射或合成失败 → 降级 `forColor`；
- `forColor`：只给一个 RGB，用 mask 生成纯色盾牌，服务于「配置里只填了颜色、没有物品贴图」的自定义盔甲。

### 4.3 材质 → 贴图路径映射

`texturePath(materialId)` 用 switch 把材质 ID 映射到胸甲（头盔）贴图，例如：

```java
case "minecraft:iron":
    return "textures/item/iron_chestplate.png";
case "minecraft:diamond":
    return "textures/item/diamond_chestplate.png";
case "minecraft:turtle":
    return "textures/item/turtle_helmet.png"; // 乌龟壳只有头盔
// 自定义材质（占位）
case "examplemod:compressed_stone_lv1":
    return "textures/block/compressed_stone/lv1.png";
default:
    return null; // 无映射 → 走纯色降级
```

`buildFromTexture` 里再按路径判断命名空间：原版贴图走 `minecraft` 命名空间，自定义贴图走模组自己的命名空间。

### 4.4 `loadMask()`：蒙版只加载一次

首次使用时读取原版 `armor_full.png`，并缓存四件东西：

1. `maskPixels[]`：全部像素（ARGB）；
2. `maskMaxLuma`：不透明像素里的最大亮度，供归一化明暗；
3. `maskMinX/MinY/MaxX/MaxY`：盾牌非透明内容的包围盒，用于把物品纹理「贴合」进盾牌轮廓。

之后所有合成都复用这张 mask，不再重复读盘。

### 4.5 `buildFromTexture()`：逐像素合成流程

这是主流程，步骤如下：

1. `loadMask()` 失败直接返回 `null`（上层降级纯色）；
2. 按命名空间定位并打开盔甲物品贴图 `NativeImage chest`；
3. `averageColor()` 统计物品贴图不透明像素的平均颜色，作为透明采样点的兜底；
4. `opaqueBBox()` 算出物品贴图非透明包围盒，裁掉四周透明边距；
5. 若有纹饰，先调用 `recoloredTrim()` 预生成纹饰像素数组；
6. **双重循环遍历 mask 每个像素**：
   - mask 透明（alpha < 128）→ 输出全透明；
   - 否则把 mask 内的相对坐标 `(relU, relV)` 线性映射到物品贴图包围盒上，采样 `baseColorAt()`；
   - 有皮革染色 `tint >= 0` → `multiply()` 做颜色乘法；
   - 有纹饰 → 在对应小区域内统计纹饰占比、取最近纹饰色，`lerp()` 线性插值叠加；
   - `shade()` 用 mask 亮度对最终颜色压暗，复刻原版盾牌的明暗体积；
7. `mirrorRightToLeft()` 右半镜像到左半（盾牌左右对称）；
8. `register()` 包装成 `DynamicTexture`。

核心循环片段（脱敏）：

```java
for (int yy = 0; yy < maskH; yy++) {
    for (int xx = 0; xx < maskW; xx++) {
        int m = maskPixels[yy * maskW + xx];
        int ma = m >>> 24 & 0xFF;
        if (ma < 128) {
            out.setPixelRGBA(xx, yy, 0);
        } else {
            // mask相对坐标 → 物品贴图包围盒坐标
            double relU = (double) (xx - maskMinX) / mbw;
            double relV = (double) (yy - maskMinY) / mbh;
            int mx = clamp(cMinX + (int) Math.round(relU * cbw), 0, cw - 1);
            int my = clamp(cMinY + (int) Math.round(relV * cbh), 0, ch - 1);

            int baseColor = baseColorAt(chest, mx, my, avg);
            if (tint >= 0) {
                baseColor = multiply(baseColor, tint);
            }

            int color = baseColor;
            if (trim != null) {
                // 区域采样纹饰占比，按权重lerp叠加（0.6 ~ 0.92）
                int tc = trimInRegion(trim, cw, rx0, ry0, x1r, y1r, mx, my);
                if (tc >= 0) {
                    tc = ArmorHudConfig.applySaturation(tc, ArmorHudConfig.get().trimSaturation());
                    int cnt = trimCount(trim, cw, rx0, ry0, x1r, y1r);
                    int total = (x1r - rx0) * (y1r - ry0);
                    double w = Math.min(0.92, Math.max(0.6, (double) cnt / total * 1.6));
                    color = lerp(baseColor, tc, w);
                }
            }
            out.setPixelRGBA(xx, yy, shade(color, m, ma));
        }
    }
}
```

### 4.6 三个颜色工具

- **`multiply(rgb, tint)`**：皮革染色。逐通道 `(原色 × 染色) / 255`，属于「正片叠底」，保留纹理明暗，只改变色相，而不是直接覆盖成纯色。
- **`lerp(a, b, w)`**：逐通道线性插值，用于纹饰与底色融合。
- **`shade(rgb, maskPx, maskAlpha)`**：取 mask 像素 luma 归一化 `f = luma / maskMaxLuma`，三通道乘 `f`，再把 mask 的 alpha 写回高位。这样合成出的图标自带原版盾牌的明暗高光。

### 4.7 纹饰（Trim）的最近邻调色板映射 `recoloredTrim()`

纹饰不是简单盖一层色，而是**复刻原版调色板映射管线**：

1. `readStrip("textures/trims/color_palettes/trim_palette.png")` 读原版参考调色板（一行色条）；
2. `readStrip(".../" + palette + ".png")` 读目标纹饰材质调色板；
3. 读胸甲纹饰模板 `textures/trims/items/chestplate_trim.png`；
4. 对模板每个不透明像素，用 `nearestIndex()` 在参考调色板里找**欧氏距离最近**的索引，再把该索引替换成目标调色板同位置颜色；
5. 输出一张与物品贴图同尺寸的纹饰数组，未命中处填 `-1`。

```java
private static int nearestIndex(int[] ref, int rgb) {
    int r = rgb >> 16 & 0xFF, g = rgb >> 8 & 0xFF, b = rgb & 0xFF;
    int best = 0;
    long bestD = Long.MAX_VALUE;
    for (int i = 0; i < ref.length; i++) {
        long dr = r - (ref[i] >> 16 & 0xFF);
        long dg = g - (ref[i] >> 8 & 0xFF);
        long db = b - (ref[i] & 0xFF);
        long d = dr * dr + dg * dg + db * db;
        if (d < bestD) { bestD = d; best = i; }
    }
    return best;
}
```

叠加时 `trimInRegion()` 在当前像素对应的小窗口里找离采样中心最近的纹饰色，`trimCount()` 统计窗口内纹饰像素占比，把占比换算成 0.6~0.92 的混合权重，避免纹饰边缘出现生硬的硬切换。

### 4.8 纯色降级 `buildFromColor()`

没有物品贴图时，直接把传入 RGB 过一遍 `shade()` 填满 mask 轮廓，生成纯色盾牌。`forColor` 连 mask 都读不到时再兜底返回原版 `MASK`，保证 HUD 永远有东西可画。

### 4.9 资源释放注意点

所有 `InputStream` 与 `NativeImage` 都在 `try/catch(Throwable)` + `addSuppressed` 的结构里关闭。`NativeImage` 是堆外显存/内存封装，不 close 会泄漏，这是动态纹理类最容易踩的坑。

> **可改进点**：当前 `register()` 对所有合成结果使用同一个注册名。若希望每个动态纹理独立存活，建议以缓存 key 派生唯一 `ResourceLocation` 注册，避免同名互相覆盖。

---

## 五、`ArmorHudRenderer`：HUD 渲染逻辑

> 脱敏包名：`package yourmod.integration.coloredarmor;`

渲染钩子由 Mixin 在原版盔甲栏绘制前调用 `renderColored(...)`：返回 `true` 表示模组接管、拦截原版；返回 `false` 则交回原版绘制。

### 5.1 布局常量

```java
private static final EquipmentSlot[] SLOT_ORDER =
        new EquipmentSlot[]{EquipmentSlot.FEET, EquipmentSlot.LEGS, EquipmentSlot.CHEST, EquipmentSlot.HEAD};
private static final int MAX_POINTS = 20;   // 最多20点护甲
private static final int ICONS = 10;        // 10个盾牌图标
private static final int ICON = 9;          // 图标9×9像素
private static final int LEFT_HALF_W = 5;    // 左半宽5，右半宽4
```

原版每点护甲占半个盾牌，20 点正好 10 个盾牌，与原版布局完全对齐。

### 5.2 `renderColored()` 主流程

```java
public static boolean renderColored(GuiGraphics g, Player player, int i, int j, int k, int l) {
    if (!ArmorHudConfig.get().hudVisible) return false;
    if (!ArmorHudConfig.get().colorArmorBar()) return false;

    int armorTotal = player.getArmorValue();
    if (armorTotal <= 0) {
        return true; // 拦截原版，但什么都不画
    }

    ArmorHudConfig cfg = ArmorHudConfig.get();
    ResourceLocation[] tex = new ResourceLocation[20];
    boolean[] enchanted = new boolean[20];
    int p = 0;

    // 靴子→护腿→胸甲→头盔，逐槽位分配护甲点
    for (EquipmentSlot slot : SLOT_ORDER) {
        ItemStack stack = player.getItemBySlot(slot);
        if (!stack.isEmpty()) {
            int points = (int) Math.round(slotArmor(stack, slot));
            if (points > 0) {
                ResourceLocation pieceTex = MaterialTextures.iconFor(
                        ArmorHudColors.materialId(stack),
                        ArmorHudColors.colorFor(stack, cfg),
                        ArmorHudColors.trimPalette(stack),
                        ArmorHudColors.leatherTint(stack)
                );
                boolean ench = stack.isEnchanted();
                for (int n = 0; n < points && p < 20; p++) {
                    tex[p] = pieceTex;
                    enchanted[p] = ench;
                    n++;
                }
                if (p >= 20) break;
            }
        }
    }

    int coloredPoints = p;
    int totalPoints = Math.max(armorTotal, coloredPoints);
    if (totalPoints > 20) totalPoints = 20;

    ResourceLocation defTex = MaterialTextures.forColor(cfg.defaultColorRGB());
    while (p < totalPoints) {
        tex[p] = defTex;
        enchanted[p] = false;
        p++;
    }
    // ……绘制10个图标
}
```

要点：

1. `hudVisible` / `colorArmorBar` 关闭 → 返回 `false` 完全交给原版；
2. 玩家 0 护甲 → 返回 `true` 拦截，避免原版重复画空栏；
3. 按 `FEET → LEGS → CHEST → HEAD` 顺序把每件盔甲的点数填进长度 20 的 `tex[]`，每个半格记录自己的纹理与是否附魔；
4. `totalPoints = max(玩家总护甲, 实际填充点数)`，超出 20 截断；不足的半格填默认色纹理，保证栏位数量与原版一致。

### 5.3 `slotArmor()`：从 AttributeModifier 读护甲值（兼容关键）

```java
private static double slotArmor(ItemStack stack, EquipmentSlot slot) {
    double[] sum = new double[]{0.0};
    stack.forEachModifier(slot, (holder, modifier) -> {
        if (modifier.operation() == Operation.ADD_VALUE && holder.value() == Attributes.ARMOR.value()) {
            sum[0] += modifier.amount();
        }
    });
    return sum[0];
}
```

没有用 `ArmorItem#getDefense()` 硬编码防御值，而是遍历该槽位的属性修饰符，只累加 `ADD_VALUE` 类型的 `minecraft:armor`。这样模组通过 attribute 动态加减护甲（宝石、套装加成、debuff）都能正确反映到 HUD 点数上。

### 5.4 半格分割渲染（核心交互）

```java
for (int o = 0; o < 10; o++) {
    int x = l + o * 8;               // 图标间距8，与原版一致
    int leftIdx = o * 2;
    int rightIdx = o * 2 + 1;
    boolean leftFilled = leftIdx < totalPoints;
    boolean rightFilled = rightIdx < totalPoints;

    if (leftFilled && rightFilled) {
        ResourceLocation tl = tex[leftIdx];
        ResourceLocation tr = tex[rightIdx];
        if (tl == tr) {
            drawArmorIcon(g, tl, x, y, x, x + 9);          // 同材质：整图一次画
        } else {
            drawArmorIcon(g, tl, x, y, x, x + 5);          // 左半：裁剪 x ~ x+5
            drawArmorIcon(g, tr, x, y, x + 5, x + 9);      // 右半：裁剪 x+5 ~ x+9
        }
    } else if (leftFilled) {
        g.blitSprite(ARMOR_EMPTY, x, y, 9, 9);             // 先铺空盾底
        drawArmorIcon(g, tex[leftIdx], x, y, x, x + 5);
    } else {
        g.blitSprite(ARMOR_EMPTY, x, y, 9, 9);
    }
    // ……流光、星光
}
```

`drawArmorIcon` 内部用 `enableScissor` 做硬裁剪：

```java
private static void drawArmorIcon(GuiGraphics g, ResourceLocation tex, int x, int y, int clipX1, int clipX2) {
    boolean clip = clipX1 > x || clipX2 < x + 9;
    if (clip) g.enableScissor(clipX1, y, clipX2, y + 9);
    g.blit(tex, x, y, 0.0F, 0.0F, 9, 9, 9, 9);
    if (clip) g.disableScissor();
}
```

同一个 9×9 盾牌，通过剪刀测试只放行左 5 像素或右 4 像素，于是一件铁靴 + 一件金胸甲凑成的 2 点护甲，就能在一个盾牌上左灰右金。

### 5.5 附魔流光 `drawEnchantGlow()`

```java
double frac = ((t * 0.025 + icon * 0.13) % 1.0 + 1.0) % 1.0;
double bandPos = frac * travel - 4.0;
RenderSystem.blendFunc(SourceFactor.SRC_ALPHA, DestFactor.ONE); // 加法混合

for (int py = 0; py < 9; py++) {
    for (int px = 0; px < 9; px++) {
        int sx = x + px;
        if (sx >= clipX1 && sx < clipX2) {
            double a = 1.0 - Math.abs(px + py - bandPos) / 4.0;
            if (a > 0.0) {
                g.setColor(gr, gg, gb, (float) (a * brightness));
                g.enableScissor(sx, sy, sx + 1, sy + 1);
                g.blitSprite(ARMOR_FULL, x, y, 9, 9);
                g.disableScissor();
            }
        }
    }
}
```

- 每个图标用 `icon * 0.13` 错开相位，光带不会同步闪烁；
- `a = 1 - |px + py - bandPos| / 4` 形成沿对角线移动的高斯状亮带，`GLOW_BAND = 4.0` 控制带宽；
- `SRC_ALPHA, ONE` 加法混合让光带在盾牌上「发亮」而不是盖一层不透明色；
- 逐像素 1px scissor + 重绘 `armor_full`，只在亮带位置着色；
- 亮度取 `cfg.glowBrightness()`，颜色经 `glowColorAdjusted()`（饱和度调整后的流光色）。

> 原代码里流光受 `cfg.stars` 总开关统一控制，关闭特效会同时停掉光带与星光。

### 5.6 动画时间 `animTime()`（防卡顿关键）

```java
private static double animTime() {
    Minecraft mc = Minecraft.getInstance();
    long gameTime = mc.level != null ? mc.level.getGameTime() : 0L;
    float partial = mc.getTimer().getGameTimeDeltaPartialTick(false);
    return (float) gameTime + partial;
}
```

动画必须用 `gameTime + partialTick`：只取整数 `gameTime` 会让动画在刻与刻之间跳帧，加上 partialTick 后高刷新率下也是平滑连续的。

### 5.7 闪烁星光 `drawStars / drawHalfStars / drawSparkle`

```java
private static void drawHalfStars(GuiGraphics g, int x, int y, int icon, int side, double t) {
    long base = mix((icon * 7L + side + 1L) * -7046029254386353131L);
    int regionX = side == 0 ? x + 1 : x + 5;
    int regionW = 3, regionY = y + 2, regionH = 5;

    for (int s = 0; s < 2; s++) {
        long h = mix(base + (s + 1L) * -4417276706812531889L);
        int sx = regionX + (int) ((h >>> 8) % regionW);
        int sy = regionY + (int) ((h >>> 16) % regionH);
        double off = (h >>> 24 & 65535L) / 65535.0 * (Math.PI * 2);
        double sine = Math.sin(t * 0.11 + off);
        double pop = sine > 0.0 ? Math.pow(sine, 10.0) : 0.0;
        double bright = Math.min(1.0, 0.1 + pop);
        int color = STAR_PALETTE[(int) (h >>> 56 & 7L) % STAR_PALETTE.length];
        drawSparkle(g, sx, sy, bright, color);
    }
}
```

- **确定性伪随机**：`mix()` 是一个整数哈希（三段 xorshift 乘常数），由「图标序号 + 左右半 + 星序号」推出位置、相位、颜色，同一图标每次刷新星点位置稳定，不会乱跳；
- 每半格 2 颗星，限制在一个 3×5 的小区域内；
- `sin^10` 让亮度只在正弦波顶端「啪」地爆一下，形成一闪而过的星光，平时维持 0.1 的微弱底亮；
- `drawSparkle` 先画 1px 白色核心，`bright > 0.5` 时再补上下左右四臂，变成十字星芒；
- `drawStars` 里 `pushPose` 后 `translate(0, 0, 250)` 把星光抬到盾牌图层之上，避免被图标盖住。

### 5.8 状态复位

每帧结束 `resetColor()` 把 `GuiGraphics` 颜色重置为白色、`disableBlend()` 关闭混合，避免污染后续原版 HUD 绘制。

---

## 六、关键技术难点归纳

| 难点 | 本方案做法 |
| --- | --- |
| 不堆资源包贴图 | 运行时读物品纹理 + mask 合成 `DynamicTexture`，带 key 缓存与负缓存 |
| 混穿半格不同材质 | `enableScissor` 把同一盾牌裁成左右两半分别 blit |
| 纹饰颜色一致 | 复刻参考调色板 → 最近邻索引 → 目标调色板的映射，并处理 `overrideArmorMaterials` |
| 皮革染色 | `multiply` 正片叠底，保留纹理明暗 |
| 改色不改变明暗 | luma 锚定的饱和度缩放算法 |
| 动态护甲值兼容 | 遍历 `ADD_VALUE` 的 `ARMOR` attribute，而非硬编码 defense |
| 动画平滑 | `gameTime + partialTick` 驱动 |
| 配置写坏不崩 | 全字段判空兜底 + 异常回退默认 + `putIfAbsent` 补默认 |
| 内存安全 | `NativeImage` / `InputStream` 全部在 Throwable 结构里关闭 |

---

## 七、如何扩展一个新的自定义盔甲材质

### 方式 A：只配颜色（不改代码）

直接在 JSON 的 `materialColors` 里加一行，模组会走 `forColor` 纯色生成，不需要任何贴图：

```json
{
  "materialColors": {
    "examplemod:my_material": "#4488FF"
  }
}
```

### 方式 B：用物品贴图合成（改代码）

1. 在 `ArmorHudConfig#defaults()` 的 `materialColors` 补默认色；
2. 在 `MaterialTextures#texturePath()` 的 switch 增加映射：

```java
case "examplemod:my_material":
    return "textures/item/my_chestplate.png";
```

3. 若贴图不在 `minecraft` 命名空间，在 `buildFromTexture()` 的命名空间判断里加上对应分支。

---

## 八、配置文件示例（脱敏）

```json
{
  "materialColors": {
    "minecraft:leather": "#8B5A2B",
    "minecraft:chainmail": "#6E6E6E",
    "minecraft:iron": "#D8D8D8",
    "minecraft:gold": "#FFD93B",
    "minecraft:diamond": "#5BE7E0",
    "minecraft:turtle": "#3FA34D",
    "minecraft:netherite": "#4B2A5A",
    "minecraft:armadillo": "#C98A5E"
  },
  "itemColors": {},
  "defaultColor": "#A8A8A8",
  "stars": true,
  "trimSaturation": 1.0,
  "glowBrightness": 0.42,
  "glowSaturation": 1.0,
  "glowColor": "#FFEEB3",
  "armorBar": true,
  "warnLowDurability": true,
  "colorArmorBar": true,
  "durabilityWarnThreshold": 0.05
}
```

---

## 九、总结

这套彩色盔甲 HUD 的核心思路区别于「预先画好一堆图标」的传统做法：

1. **运行时合成动态纹理**，读盔甲物品贴图套盾牌 mask，对模组自定义盔甲兼容性强；
2. **完整复刻 1.20+ 纹饰调色板管线**，并处理材质覆写，HUD 与实体颜色一致；
3. **scissor 半格分割**，混穿盔甲也能一盾双色；
4. **JSON 三级颜色回退**（物品 > 材质 > 默认），配置驱动；
5. 附魔光带用加法混合 + 相位错峰，星光用确定性哈希 + `sin^10` 爆闪，特效全部可调；
6. 大量防御性代码：负缓存、IO 容错、NPE 兜底、资源关闭，兼顾性能与稳定。

> 配套还需要一个 Mixin 钩子：在原版 HUD 盔甲栏绘制处调用 `ArmorHudRenderer.renderColored()`，并在返回 `true` 时取消原版绘制；其余三个类即可独立工作。
