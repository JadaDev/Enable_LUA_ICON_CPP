# Enabling GetItemIcon Function in LUA (Latest as of 2024/07/04)

This guide explains how to add the GetItemIcon function in LUA for TrinityCore. Credits to Rocket2 & Artamedes for their assistance!

## Instructions

### Step 1: Modify GlobalMethods.h

1. Open `src/server/game/LuaEngine/methods/TrinityCore/GlobalMethods.h`.
2. Locate "GetItemLink" and add the following code after Line 400:

    ```cpp
    int GetItemIcon(Eluna* E)
    {
        uint32 entry = E->CHECKVAL<uint32>(1);
        uint8 locale = E->CHECKVAL<uint8>(2, DEFAULT_LOCALE);
        if (locale >= TOTAL_LOCALES)
            return luaL_argerror(E->L, 2, "valid LocaleConstant expected");

        const ItemTemplate* temp = eObjectMgr->GetItemTemplate(entry);
        std::ostringstream ss;
        ss << "|TInterface";

        const ItemDisplayInfoEntry* dispInfo = NULL;
        if (temp)
        {
            dispInfo = sItemDisplayInfoStore.LookupEntry(temp->DisplayInfoID);
            if (dispInfo)
                ss << "/ICONS/" << dispInfo->InventoryIcon[0];
        }
        if (!dispInfo)
            ss << "/InventoryItems/WoWUnknownItem01";
        E->Push(ss.str());
        return 1;
    }
    ```

### Step 2: Update GlobalMethods.h

1. Scroll to the bottom and add the following line after Line 3271:

    ```cpp
    { "GetItemIcon", &LuaGlobalFunctions::GetItemIcon },
    ```

### Step 3: Modify DBCStores.cpp

1. Update Line 118 to:

    ```cpp
    DBCStorage<ItemDisplayInfoEntry> sItemDisplayInfoStore(ItemDisplayTemplateEntryfmt);
    ```

### Step 4: Modify DBCStores.h

1. Update Line 148 to:

    ```cpp
    extern DBCStorage<ItemDisplayInfoEntry> sItemDisplayInfoStore;
    ```

### Step 5: Modify DBCStructure.h

1. Uncomment lines 900 to 914 to include `ItemDisplayInfoEntry` struct:

    ```cpp
    struct ItemDisplayInfoEntry
    {
        uint32 ID;                                              // 0
        char const* ModelName[2];                               // 1-2
        char const* ModelTexture[2];                            // 3-4
        char const* InventoryIcon[2];                           // 5-6
        uint32 GeosetGroup[3];                                  // 7-9
        uint32 Flags;                                           // 10
        uint32 SpellVisualID;                                   // 11
        uint32 GroupSoundIndex;                                 // 12
        uint32 HelmetGeosetVisID[2];                            // 13-14
        char const* Texture[8];                                 // 15-22
        int32 ItemVisual;                                       // 23
        uint32 ParticleColorID;                                 // 24
    };
    ```

### Step 6: Modify DBCfmt.h

1. Update Line 79 to:

    ```cpp
    constexpr char ItemDisplayTemplateEntryfmt[] = "nssssssiiiiiiiissssssssii";
    ```

### Step 7: Recompile TrinityCore

### Step 8: Usage

- Use `GetItemIcon(ItemEntry)` in LUA scripts to retrieve the item icon.
- Append `":30:30:-20:0|t"` for proper display.

---

This guide provides detailed steps to integrate the GetItemIcon function into your TrinityCore server setup.
