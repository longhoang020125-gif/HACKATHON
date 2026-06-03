# Tasks: Voice-to-Cart Backend Implementation

## Task 1  Project Scaffolding
- [x] Create solution and project: `dotnet new webapi -n VoiceToCart.Api`
- [x] Add `Microsoft.AspNetCore.OpenApi` (included by default in .NET 8)
- [x] Configure `appsettings.json` with `DataFilePaths:Merchants`
- [x] Configure CORS in `Program.cs`
- [x] Register all DI services in `Program.cs`

## Task 2  Data File
- [x] Copy extracted `merchants.json` into `Data/` folder
- [x] Set `CopyToOutputDirectory = Always` in `.csproj`

## Task 3  Merchant Domain Models
- [x] `Models/Merchant/Merchant.cs`
- [x] `Models/Merchant/MenuItem.cs`
- [x] `Models/Merchant/OptionGroup.cs`
- [x] `Models/Merchant/MenuOption.cs`
- [x] `Models/Merchant/MerchantsRoot.cs` (wrapper: `{ "merchants": [...] }`)

## Task 4  Cart Models
- [x] `Models/Cart/CartItem.cs`
- [x] `Models/Cart/CartSession.cs`

## Task 5  DTOs
- [x] `Models/Dto/VoiceCommandRequest.cs`
- [x] `Models/Dto/VoiceCommandResult.cs`
- [x] `Models/Dto/MenuItemSuggestion.cs`
- [x] `Models/Dto/UpdateCartItemRequest.cs`

## Task 6  Interfaces
- [x] `Services/Interfaces/IMerchantRepository.cs`
- [x] `Services/Interfaces/ICartSessionStore.cs`

## Task 7  MockMerchantRepository
- [x] Load JSON from path in `IConfiguration` on startup
- [x] Implement `FindItemByKeyword` with diacritic normalization + substring match
- [x] Implement `GetAllMerchants`

## Task 8  InMemoryCartSessionStore
- [x] `ConcurrentDictionary` backing store
- [x] `GetOrCreate`, `Get`, `Save` implementations

## Task 9  VoiceCartService
- [x] `ProcessVoiceCommand`: match  accurate/ambiguous/no-result logic
- [x] `UpdateCartItem`: correction path (update qty, notes, delete)

## Task 10  CartController
- [x] `POST /api/cart/voice-command`
- [x] `PUT /api/cart/item`
- [x] `GET /api/cart`
- [x] Validation and error responses (400, 404)

## Task 11  Verification
- [x] `dotnet build` passes with no errors
- [x] Manual smoke test via Swagger UI
