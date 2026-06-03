# Design: Voice-to-Cart Backend

## 1. Project Structure

```
VoiceToCart.Api/
 Data/
    merchants.json
 Models/
    Merchant/
       Merchant.cs
       MenuItem.cs
       OptionGroup.cs
       MenuOption.cs
    Cart/
       CartSession.cs
       CartItem.cs
    Dto/
        VoiceCommandRequest.cs
        VoiceCommandResult.cs
        UpdateCartItemRequest.cs
        MenuItemSuggestion.cs
 Services/
    Interfaces/
       IMerchantRepository.cs
       ICartSessionStore.cs
    MockMerchantRepository.cs
    InMemoryCartSessionStore.cs
    VoiceCartService.cs
 Controllers/
    CartController.cs
 Program.cs
 appsettings.json
```

---

## 2. Data Models

### 2.1 Merchant Domain Models

```csharp
// Models/Merchant/Merchant.cs
public class Merchant {
    public string MerchantId { get; set; }
    public string MerchantName { get; set; }
    public string CuisineType { get; set; }
    public string Address { get; set; }
    public List<MenuItem> Menu { get; set; }
}

// Models/Merchant/MenuItem.cs
public class MenuItem {
    public string ItemId { get; set; }
    public string ItemName { get; set; }
    public List<string> Aliases { get; set; }
    public int BasePrice { get; set; }
    public List<OptionGroup> OptionGroups { get; set; }
}

// Models/Merchant/OptionGroup.cs
public class OptionGroup {
    public string GroupName { get; set; }
    public bool IsMandatory { get; set; }
    public List<MenuOption> Options { get; set; }
}

// Models/Merchant/MenuOption.cs
public class MenuOption {
    public string OptionId { get; set; }
    public string OptionName { get; set; }
    public int AdditionalPrice { get; set; }
}
```

### 2.2 Cart Models

```csharp
// Models/Cart/CartItem.cs
public class CartItem {
    public string CartItemId { get; set; }       // GUID generated on add
    public string ItemId { get; set; }            // ref to MenuItem.ItemId
    public string MerchantId { get; set; }
    public string ItemName { get; set; }
    public int Quantity { get; set; }
    public int UnitPrice { get; set; }            // base_price + selected option prices
    public string SpecialNotes { get; set; }      // e.g. "khong hanh"
    public List<string> SelectedOptionIds { get; set; }
}

// Models/Cart/CartSession.cs
public class CartSession {
    public string SessionId { get; set; }
    public List<CartItem> Items { get; set; }
    public int TotalPrice => Items.Sum(i => i.UnitPrice * i.Quantity);
    public DateTime LastUpdated { get; set; }
}
```

### 2.3 DTOs

```csharp
// VoiceCommandRequest - incoming from frontend (LLM-parsed)
{ sessionId, dishName, quantity, specialNotes }

// VoiceCommandResult - outgoing response
{ isAmbiguous, message, cart: CartSession, suggestions: List<MenuItemSuggestion> }

// MenuItemSuggestion - for ambiguous results
{ itemId, merchantId, merchantName, itemName, basePrice, aliases }

// UpdateCartItemRequest - for PUT /api/cart/item
{ sessionId, cartItemId, quantity, specialNotes }
```

---

## 3. Service Layer

### 3.1 IMerchantRepository
```
FindItemByKeyword(string keyword) -> List<(Merchant, MenuItem)>
GetAllMerchants() -> List<Merchant>
```

### 3.2 MockMerchantRepository
- Constructor: inject `IConfiguration` to read file path.
- On startup: `JsonSerializer.Deserialize<MerchantsRoot>(File.ReadAllText(path))`.
- `FindItemByKeyword`: iterate all merchants/menu items, normalize both keyword and
  item names/aliases (strip Vietnamese diacritics via Unicode normalization), return
  all pairs where either `item_name` or any alias contains the normalized keyword.

### 3.3 ICartSessionStore
```
GetOrCreate(string sessionId) -> CartSession
Get(string sessionId) -> CartSession?
Save(CartSession session) -> void
```

### 3.4 InMemoryCartSessionStore
- Uses `ConcurrentDictionary<string, CartSession>` as backing store.
- `GetOrCreate`: if key missing, initialize new `CartSession`.

### 3.5 VoiceCartService
```
ProcessVoiceCommand(VoiceCommandRequest) -> VoiceCommandResult
UpdateCartItem(UpdateCartItemRequest) -> CartSession
```

**ProcessVoiceCommand logic:**
1. `session = sessionStore.GetOrCreate(request.SessionId)`
2. `matches = merchantRepo.FindItemByKeyword(request.DishName)`
3. If `matches.Count == 1`:
   - Check if item already in cart (same `ItemId`): if yes, increment quantity.
   - Else add new `CartItem` (new GUID, copy fields, set UnitPrice = BasePrice).
   - Return `{ IsAmbiguous=false, Cart=session }`.
4. If `matches.Count >= 2`:
   - Return `{ IsAmbiguous=true, Suggestions=[...], Cart=session }`.
5. If `matches.Count == 0`:
   - Return `{ IsAmbiguous=false, Message="Khong tim thay mon phu hop", Cart=session }`.

**UpdateCartItem logic:**
1. `session = sessionStore.Get(request.SessionId)`  404 if null.
2. Find `CartItem` by `CartItemId`  404 if null.
3. If `quantity == 0`: remove item.
4. Else: update quantity and/or specialNotes.
5. Save and return session.

---

## 4. API Controller

### CartController  route prefix `/api/cart`

| Method | Route           | Handler                        | Returns             |
|--------|-----------------|-------------------------------|---------------------|
| POST   | /voice-command  | ProcessVoiceCommand            | VoiceCommandResult  |
| PUT    | /item           | UpdateCartItem                 | CartSession         |
| GET    | /               | GetCart(?sessionId=)           | CartSession         |

---

## 5. Vietnamese Diacritic Normalization

Use `string.Normalize(NormalizationForm.FormD)` + regex to strip `\p{M}` combining marks, then lowercase. This maps "Phở Bò"  "pho bo", enabling matching against "pho bo" keyword.

---

## 6. DI Registration (Program.cs)

```csharp
builder.Services.AddSingleton<IMerchantRepository, MockMerchantRepository>();
builder.Services.AddSingleton<ICartSessionStore, InMemoryCartSessionStore>();
builder.Services.AddScoped<VoiceCartService>();
```
