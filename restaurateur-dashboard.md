# Documentation API du Dashboard Restaurateur

## 1. Vue d'ensemble du Dashboard

Le dashboard restaurateur permet de :
- Visualiser les statistiques du jour
- Gérer les disponibilités
- Suivre les commandes en cours
- Gérer le menu et les plats
- Consulter les revenus

## 2. Endpoints API

### 2.1 Statistiques du Dashboard
```http
GET /api/restaurants/{restaurantId}/dashboard/stats
```

**Réponse :**
```typescript
interface DashboardStats {
  activeDishes: number;          // Nombre de plats actifs
  todayOrders: number;          // Nombre de commandes du jour
  monthlyRevenue: number;       // Revenus du mois
  averageRating: number;        // Note moyenne
  servedCompanies: number;      // Nombre d'entreprises servies
  orderDeadline: string;        // Heure limite de commande (HH:mm)
  availabilityStatus: boolean;  // Statut de disponibilité global
}
```

### 2.2 Gestion des Disponibilités

#### Obtenir les horaires
```http
GET /api/restaurants/{restaurantId}/schedules
```

**Réponse :**
```typescript
interface RestaurantSchedule {
  id: string;                  // UUID
  restaurant_id: string;       // UUID
  organization_id: string;     // UUID
  day_id: string;             // UUID
  is_available: boolean;
  note: string | null;
  is_default: boolean;
  order_deadline: string;      // HH:mm
  created_at: string;
  updated_at: string;
}
```

#### Mettre à jour les horaires
```http
PUT /api/restaurants/{restaurantId}/schedules
```

**Corps de la requête :**
```typescript
interface UpdateScheduleRequest {
  schedules: Array<{
    id?: string;              // UUID, optionnel pour création
    organization_id: string;  // UUID
    day_id: string;          // UUID
    is_available: boolean;
    order_deadline: string;   // HH:mm
    note?: string;
    is_default?: boolean;
  }>;
}
```

### 2.3 Gestion des Plats

#### Liste des plats
```http
GET /api/restaurants/{restaurantId}/dishes
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
  created_at: string;
  updated_at: string;
}
```

#### Prix des plats
```http
GET /api/restaurants/{restaurantId}/dish-prices
```

**Réponse :**
```typescript
interface DishPrice {
  id: string;           // UUID
  restaurant_id: string; // UUID
  dish_id: string;      // UUID
  price: number;        // Decimal(10,2)
}
```

### 2.4 Gestion des Commandes

#### Liste des commandes du jour
```http
GET /api/restaurants/{restaurantId}/orders/today
```

**Paramètres de requête :**
```typescript
interface OrdersQueryParams {
  status?: 'pending' | 'processing' | 'completed' | 'cancelled';
  organization_id?: string;  // UUID
}
```

**Réponse :**
```typescript
interface Order {
  id: string;                // UUID
  user_id: string;          // UUID
  restaurant_id: string;    // UUID
  order_status_id: string;  // UUID
  total_amount: number;     // Decimal(10,2)
  created_at: string;
  items: Array<{
    id: string;            // UUID
    dish_id: string;      // UUID
    quantity: number;
    unit_price: number;   // Decimal(10,2)
    supplements: Array<{
      id: string;         // UUID
      name: string;
      price: number;      // Decimal(5,2)
    }>;
  }>;
}
```

### 2.5 Statistiques des Revenus

#### Revenus par période
```http
GET /api/restaurants/{restaurantId}/revenue
```

**Paramètres de requête :**
```typescript
interface RevenueQueryParams {
  period: 'day' | 'week' | 'month' | 'year';
  start_date: string;    // YYYY-MM-DD
  end_date: string;      // YYYY-MM-DD
}
```

**Réponse :**
```typescript
interface Revenue {
  total: number;         // Decimal(10,2)
  breakdown: Array<{
    date: string;        // YYYY-MM-DD
    amount: number;      // Decimal(10,2)
    order_count: number;
  }>;
}
```

## 3. Codes d'Erreur Spécifiques

```typescript
enum RestaurateurErrorCodes {
  SCHEDULE_CONFLICT = 'SCHEDULE_CONFLICT',        // Conflit d'horaires
  INVALID_DEADLINE = 'INVALID_DEADLINE',          // Heure limite invalide
  DISH_NOT_FOUND = 'DISH_NOT_FOUND',             // Plat non trouvé
  INVALID_PRICE = 'INVALID_PRICE',               // Prix invalide
  ORDER_STATUS_CHANGE_DENIED = 'ORDER_STATUS_CHANGE_DENIED' // Changement de statut non autorisé
}
``` 