# Documentation API de la Gestion des Disponibilités

## 1. Vue d'ensemble

La section "Disponibilités" permet de :
- Gérer le planning hebdomadaire par défaut
- Configurer les disponibilités par organisation
- Définir les heures limites de commande
- Définir les heures de livraison
- Gérer les jours d'indisponibilité avec notes
- Dupliquer les plannings entre semaines
- Appliquer un planning par défaut à toutes les organisations

## 2. Endpoints API

### 2.1 Gestion des Horaires

#### Obtenir les horaires d'un restaurant
```http
GET /api/restaurants/{restaurantId}/schedules
```

**Paramètres de requête :**
```typescript
interface SchedulesQueryParams {
  week_number?: number;     // 1-4 pour les semaines du mois
  organization_id?: string; // UUID, optionnel pour filtrer par organisation
}
```

**Réponse :**
```typescript
interface RestaurantSchedule {
  id: string;              // UUID
  restaurant_id: string;   // UUID
  organization_id: string | null;  // null = planning par défaut
  day_id: string;          // UUID
  order_deadline: string;  // Format "HH:mm"
  delivery_time: string;   // Format "HH:mm"
  is_available: boolean;
  note: string | null;     // Note obligatoire si indisponible
  is_default: boolean;     // true si c'est le planning par défaut
  created_at: string;
  updated_at: string;
}
```

#### Mettre à jour un horaire
```http
PUT /api/restaurants/{restaurantId}/schedules/{scheduleId}
```

**Corps de la requête :**
```typescript
interface UpdateScheduleRequest {
  order_deadline?: string;  // Format "HH:mm"
  delivery_time?: string;   // Format "HH:mm"
  is_available?: boolean;
  note?: string;           // Requis si is_available = false
}
```

### 2.2 Gestion des Jours

#### Liste des jours de la semaine
```http
GET /api/days
```

**Réponse :**
```typescript
interface Day {
  id: string;              // UUID
  name: string;            // Nom complet (ex: "Lundi")
  shortName: string;       // Nom court (ex: "Lun")
  created_at: string;
}
```

### 2.3 Gestion des Organisations

#### Liste des organisations d'un restaurant
```http
GET /api/restaurants/{restaurantId}/organizations
```

**Réponse :**
```typescript
interface Organization {
  id: string;              // UUID
  name: string;
  contact: string | null;
  full_address: string | null;
  created_at: string;
  updated_at: string;
}
```

### 2.4 Duplication des Plannings

#### Dupliquer un planning vers une autre semaine
```http
POST /api/restaurants/{restaurantId}/schedules/duplicate
```

**Corps de la requête :**
```typescript
interface DuplicateScheduleRequest {
  source_week: number;     // 1-4
  target_week: number;     // 1-4
  organization_id?: string; // UUID, optionnel
}
```

#### Appliquer le planning par défaut à toutes les organisations
```http
POST /api/restaurants/{restaurantId}/schedules/apply-default
```

**Corps de la requête :**
```typescript
interface ApplyDefaultScheduleRequest {
  organization_ids: string[]; // Liste des UUIDs des organisations
  week_number: number;       // 1-4
}
```

### 2.5 Validation des Horaires

#### Vérifier les conflits d'horaires
```http
POST /api/restaurants/{restaurantId}/schedules/validate
```

**Corps de la requête :**
```typescript
interface ValidateScheduleRequest {
  schedules: Array<{
    day_id: string;        // UUID
    order_deadline: string; // Format "HH:mm"
    delivery_time: string;  // Format "HH:mm"
  }>;
}
```

**Réponse :**
```typescript
interface ValidationResponse {
  isValid: boolean;
  conflicts: Array<{
    day_id: string;
    message: string;
  }>;
}
```

## 3. Codes d'Erreur Spécifiques

```typescript
enum ScheduleErrorCodes {
  SCHEDULE_NOT_FOUND = 'SCHEDULE_NOT_FOUND',     // Horaire non trouvé
  INVALID_TIME_FORMAT = 'INVALID_TIME_FORMAT',   // Format d'heure invalide
  MISSING_NOTE = 'MISSING_NOTE',                 // Note manquante pour un jour indisponible
  INVALID_WEEK_NUMBER = 'INVALID_WEEK_NUMBER',   // Numéro de semaine invalide
  DUPLICATION_FAILED = 'DUPLICATION_FAILED',     // Échec de la duplication
  CONFLICTING_SCHEDULE = 'CONFLICTING_SCHEDULE', // Conflit d'horaires
  ORGANIZATION_NOT_FOUND = 'ORGANIZATION_NOT_FOUND', // Organisation non trouvée
  INVALID_DEADLINE = 'INVALID_DEADLINE',         // Heure limite invalide
  INVALID_DELIVERY_TIME = 'INVALID_DELIVERY_TIME' // Heure de livraison invalide
}
```

## 4. Règles de Validation

1. **Notes obligatoires** :
   - Une note est requise pour chaque jour marqué comme indisponible
   - La note doit contenir au moins 5 caractères

2. **Heures de commande** :
   - L'heure limite de commande doit être antérieure à l'heure de livraison
   - L'heure limite de commande ne peut pas être plus de 24h avant l'heure de livraison

3. **Plannings par défaut** :
   - Un restaurant doit avoir un planning par défaut
   - Le planning par défaut s'applique à toutes les organisations sauf si un planning spécifique est défini

4. **Duplication** :
   - La duplication est possible uniquement vers des semaines futures
   - La duplication conserve les notes et les statuts d'indisponibilité 