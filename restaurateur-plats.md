# Documentation API de la Gestion des Plats

## 1. Vue d'ensemble

La section "Mes Plats" permet de :
- Gérer le catalogue des plats
- Organiser les plats par catégories
- Gérer les suppléments et leurs prix
- Suivre les statistiques de vente
- Gérer les allergènes et les tags
- Exporter les données des plats
- Utiliser des modèles prédéfinis
- Gérer la disponibilité des plats

## 2. Endpoints API

### 2.1 Gestion des Plats

#### Liste des plats
```http
GET /api/restaurants/{restaurantId}/dishes
```

**Paramètres de requête :**
```typescript
interface DishesQueryParams {
  page?: number;
  limit?: number;
  search?: string;
  category?: string;
  sort_by?: 'name' | 'price' | 'popularity';
  sort_order?: 'asc' | 'desc';
  available_only?: boolean;
}
```

**Réponse :**
```typescript
interface Dish {
  id: string;                // UUID
  name: string;
  description: string | null;
  image_url: string | null;
  tags: string[];
  category_id: string | null; // UUID
  restaurant_id: string;     // UUID
  is_cloned: boolean;
  is_preset: boolean;
  available: boolean;
  order_count: number;
  allergens: string[];
  supplements: Array<{
    id: string;             // UUID
    name: string;
    price: number;          // Decimal(5,2)
  }>;
  created_at: string;
  updated_at: string;
}
```

#### Créer un plat
```http
POST /api/restaurants/{restaurantId}/dishes
```

**Corps de la requête :**
```typescript
interface CreateDishRequest {
  name: string;
  description: string;
  image_url?: string;
  category_id?: string;     // UUID
  available: boolean;
  allergens: string[];
  supplements: Array<{
    id: string;            // UUID
    price: number;         // Decimal(5,2)
  }>;
  tags: string[];
}
```

#### Mettre à jour un plat
```http
PUT /api/restaurants/{restaurantId}/dishes/{dishId}
```

**Corps de la requête :**
```typescript
interface UpdateDishRequest {
  name?: string;
  description?: string;
  image_url?: string;
  category_id?: string;     // UUID
  available?: boolean;
  allergens?: string[];
  supplements?: Array<{
    id: string;            // UUID
    price: number;         // Decimal(5,2)
  }>;
  tags?: string[];
}
```

#### Supprimer un plat
```http
DELETE /api/restaurants/{restaurantId}/dishes/{dishId}
```

### 2.2 Gestion des Catégories

#### Liste des catégories
```http
GET /api/restaurants/{restaurantId}/categories
```

**Réponse :**
```typescript
interface Category {
  id: string;              // UUID
  name: string;
  description: string | null;
  shortcut_label: string | null;  // Emoji ou raccourci visuel
  created_at: string;
  updated_at: string;
}
```

#### Créer une catégorie
```http
POST /api/restaurants/{restaurantId}/categories
```

**Corps de la requête :**
```typescript
interface CreateCategoryRequest {
  name: string;
  description?: string;
  shortcut_label?: string;
}
```

#### Mettre à jour une catégorie
```http
PUT /api/restaurants/{restaurantId}/categories/{categoryId}
```

**Corps de la requête :**
```typescript
interface UpdateCategoryRequest {
  name?: string;
  description?: string;
  shortcut_label?: string;
}
```

#### Supprimer une catégorie
```http
DELETE /api/restaurants/{restaurantId}/categories/{categoryId}
```

### 2.3 Gestion des Suppléments

#### Liste des suppléments
```http
GET /api/restaurants/{restaurantId}/supplements
```

**Réponse :**
```typescript
interface Supplement {
  id: string;              // UUID
  name: string;
  preset_group: string | null;
  created_at: string;
  updated_at: string;
}
```

#### Prix des suppléments par restaurant
```http
GET /api/restaurants/{restaurantId}/supplement-prices
```

**Réponse :**
```typescript
interface SupplementPrice {
  id: string;              // UUID
  restaurant_id: string;   // UUID
  supplement_id: string;   // UUID
  price: number;           // Decimal(5,2)
}
```

#### Mettre à jour le prix d'un supplément
```http
PUT /api/restaurants/{restaurantId}/supplement-prices/{supplementId}
```

**Corps de la requête :**
```typescript
interface UpdateSupplementPriceRequest {
  price: number;           // Decimal(5,2)
}
```

### 2.4 Statistiques des Plats

#### Statistiques globales
```http
GET /api/restaurants/{restaurantId}/dishes/stats
```

**Réponse :**
```typescript
interface DishStats {
  total_dishes: number;
  available_dishes: number;
  total_revenue: number;    // Decimal(10,2)
  popular_dishes: Array<{
    id: string;            // UUID
    name: string;
    order_count: number;
    revenue: number;       // Decimal(10,2)
  }>;
}
```

### 2.5 Gestion des Allergènes

#### Liste des allergènes
```http
GET /api/allergens
```

**Réponse :**
```typescript
interface Allergen {
  id: string;              // UUID
  name: string;
  created_at: string;
}
```

#### Associer un allergène à un plat
```http
POST /api/restaurants/{restaurantId}/dishes/{dishId}/allergens
```

**Corps de la requête :**
```typescript
interface AddAllergenRequest {
  allergen_id: string;     // UUID
}
```

### 2.6 Gestion des Tags

#### Liste des tags
```http
GET /api/restaurants/{restaurantId}/tags
```

**Réponse :**
```typescript
interface Tag {
  id: string;              // UUID
  name: string;
  color: string;           // Format hexadécimal (#RRGGBB)
  created_at: string;
}
```

#### Créer un tag
```http
POST /api/restaurants/{restaurantId}/tags
```

**Corps de la requête :**
```typescript
interface CreateTagRequest {
  name: string;
  color: string;           // Format hexadécimal (#RRGGBB)
}
```

#### Associer un tag à un plat
```http
POST /api/restaurants/{restaurantId}/dishes/{dishId}/tags
```

**Corps de la requête :**
```typescript
interface AddTagRequest {
  tag_id: string;          // UUID
}
```

### 2.7 Export des Plats

#### Exporter les plats
```http
GET /api/restaurants/{restaurantId}/dishes/export
```

**Paramètres de requête :**
```typescript
interface ExportQueryParams {
  format?: 'csv' | 'excel';
  include_stats?: boolean;
  date_range?: {
    start: string;         // Format ISO
    end: string;          // Format ISO
  };
}
```

## 3. Codes d'Erreur Spécifiques

```typescript
enum DishErrorCodes {
  DISH_NOT_FOUND = 'DISH_NOT_FOUND',           // Plat non trouvé
  CATEGORY_NOT_FOUND = 'CATEGORY_NOT_FOUND',   // Catégorie non trouvée
  SUPPLEMENT_NOT_FOUND = 'SUPPLEMENT_NOT_FOUND', // Supplément non trouvé
  INVALID_PRICE = 'INVALID_PRICE',             // Prix invalide
  INVALID_IMAGE = 'INVALID_IMAGE',             // Image invalide
  DUPLICATE_DISH = 'DUPLICATE_DISH',           // Plat en double
  INVALID_CATEGORY = 'INVALID_CATEGORY',       // Catégorie invalide
  TAG_NOT_FOUND = 'TAG_NOT_FOUND',            // Tag non trouvé
  INVALID_TAG_COLOR = 'INVALID_TAG_COLOR',     // Couleur de tag invalide
  EXPORT_FAILED = 'EXPORT_FAILED',             // Échec de l'export
  INVALID_SUPPLEMENT_PRICE = 'INVALID_SUPPLEMENT_PRICE', // Prix de supplément invalide
  IMAGE_TOO_LARGE = 'IMAGE_TOO_LARGE',        // Image trop volumineuse
  INVALID_IMAGE_FORMAT = 'INVALID_IMAGE_FORMAT' // Format d'image non supporté
}
```

## 4. Règles de Validation

1. **Plats** :
   - Le nom est obligatoire et doit contenir entre 3 et 100 caractères
   - La description est obligatoire et doit contenir au moins 10 caractères
   - Le prix doit être positif et inférieur à 1000€
   - L'image ne doit pas dépasser 5MB et doit être au format JPG/PNG

2. **Catégories** :
   - Le nom est obligatoire et unique par restaurant
   - Le shortcut_label doit être un emoji ou un texte court (max 2 caractères)

3. **Suppléments** :
   - Le prix doit être positif et inférieur à 50€
   - Un supplément ne peut pas être associé plus de 10 fois au même plat

4. **Tags** :
   - Le nom est obligatoire et unique par restaurant
   - La couleur doit être au format hexadécimal valide
   - Un plat ne peut pas avoir plus de 5 tags

5. **Allergènes** :
   - Un plat ne peut pas avoir plus de 10 allergènes
   - Les allergènes doivent être dans la liste prédéfinie

6. **Export** :
   - La période d'export ne peut pas dépasser 1 an
   - Les statistiques ne sont disponibles que pour les 3 derniers mois 