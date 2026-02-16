# Business Rules and Requirements

## 👤 User Management

### User Entity Rules

**Attributes**:
- ✅ `first_name`: Required, string, 1-50 characters
- ✅ `last_name`: Required, string, 1-50 characters
- ✅ `email`: Required, unique, valid email format
- ✅ `password`: Required, minimum 8 characters, hashed
- ✅ `is_admin`: Boolean, default false
- ✅ `id`: UUID, auto-generated
- ✅ `created_at`: DateTime, auto-generated
- ✅ `updated_at`: DateTime, auto-updated

**Operations**:
1. **Register**
   - Email must be unique
   - Password must be hashed before storage
   - Cannot register as admin directly

2. **Update Profile**
   - Users can update their own profile
   - Admins can update any profile
   - Email changes require re-verification

3. **Delete Account**
   - Users can delete their own account
   - Admins can delete any account
   - Cascade delete user's places and reviews

4. **Authentication**
   - Email and password required
   - Password must match hashed value
   - Return JWT token on success

**Validation Rules**:
- Email format: `user@domain.com`
- Password: min 8 chars, 1 uppercase, 1 number
- Names: alphabetic only, no special characters

---

## 🏠 Place Management

### Place Entity Rules

**Attributes**:
- ✅ `title`: Required, string, 1-100 characters
- ✅ `description`: Optional, string, max 1000 characters
- ✅ `price`: Required, decimal > 0
- ✅ `latitude`: Required, decimal, -90 to 90
- ✅ `longitude`: Required, decimal, -180 to 180
- ✅ `owner_id`: Required, valid User UUID
- ✅ `id`: UUID, auto-generated
- ✅ `created_at`: DateTime, auto-generated
- ✅ `updated_at`: DateTime, auto-updated

**Operations**:
1. **Create**
   - User must be authenticated
   - All required fields must be present
   - Coordinates must be valid

2. **Update**
   - Only owner or admin can update
   - Cannot change owner_id
   - Price must remain > 0

3. **Delete**
   - Only owner or admin can delete
   - Cascade delete reviews
   - Remove amenity associations

4. **List**
   - Public endpoint (no auth required)
   - Support filtering and pagination
   - Return with owner info

**Validation Rules**:
- Price: positive decimal, max 2 decimals
- Coordinates: valid geographic coordinates
- Title: no offensive language
- Owner must exist in system

**Amenity Association**:
- Many-to-many relationship
- Can add/remove amenities
- Amenities must exist before association

---

## ⭐ Review Management

### Review Entity Rules

**Attributes**:
- ✅ `place_id`: Required, valid Place UUID
- ✅ `user_id`: Required, valid User UUID
- ✅ `rating`: Required, integer, 1-5
- ✅ `comment`: Required, string, 10-1000 characters
- ✅ `id`: UUID, auto-generated
- ✅ `created_at`: DateTime, auto-generated
- ✅ `updated_at`: DateTime, auto-updated

**Operations**:
1. **Create**
   - User must be authenticated
   - Place must exist
   - User cannot review their own place
   - One review per user per place

2. **Update**
   - Only author or admin can update
   - Cannot change place_id or user_id
   - Rating must remain 1-5

3. **Delete**
   - Only author or admin can delete

4. **List by Place**
   - Public endpoint
   - Sort by date (newest first)
   - Include user info

**Validation Rules**:
- Rating: integer between 1 and 5
- Comment: minimum 10 characters
- No offensive language
- Place and user must exist

**Business Constraints**:
- Users cannot review their own properties
- Only one review per user per place
- Reviews can be updated within 7 days of creation

---

## 🛋️ Amenity Management

### Amenity Entity Rules

**Attributes**:
- ✅ `name`: Required, string, unique, 1-50 characters
- ✅ `description`: Optional, string, max 200 characters
- ✅ `id`: UUID, auto-generated
- ✅ `created_at`: DateTime, auto-generated
- ✅ `updated_at`: DateTime, auto-updated

**Operations**:
1. **Create**
   - Admin only
   - Name must be unique
   - Standardized naming (e.g., "WiFi", not "wifi" or "Wi-Fi")

2. **Update**
   - Admin only
   - Name must remain unique
   - Cannot update if associated with places

3. **Delete**
   - Admin only
   - Cannot delete if associated with places
   - Must remove associations first

4. **List**
   - Public endpoint
   - Alphabetically sorted
   - Show usage count

**Validation Rules**:
- Name: alphanumeric and spaces only
- Description: no offensive language
- Name uniqueness (case-insensitive)

**Common Amenities**:
- WiFi
- Parking
- Air Conditioning
- Kitchen
- Washer/Dryer
- Pool
- Gym
- Pet Friendly

---

## 🔐 Authorization Rules

### Role-Based Access Control

**Regular Users Can**:
- Register and manage their account
- Create, update, delete their places
- Create, update, delete their reviews
- View all public content

**Administrators Can**:
- All regular user operations
- Manage any user account
- Manage any place
- Manage any review
- Create, update, delete amenities
- View system statistics

### Ownership Rules

- Users own their places
- Users own their reviews
- Only owners or admins can modify owned content
- Deletion of user cascades to owned content

---

## 📊 Data Integrity Rules

### UUID Generation
- All entities use UUID v4
- Generated at creation time
- Immutable after creation

### Timestamps
- `created_at`: Set once at creation
- `updated_at`: Updated on every modification
- ISO 8601 format
- UTC timezone

### Cascade Deletion
- User deletion → Delete all owned places and reviews
- Place deletion → Delete all associated reviews
- Amenity dissociation before deletion

### Referential Integrity
- Foreign keys must reference existing records
- No orphaned records
- Database constraints enforced

---

## 🎯 Validation Hierarchy
```
1. API Layer: Format validation
   - Required fields present
   - Data types correct
   - Basic format rules

2. Facade Layer: Business rule validation
   - User permissions
   - Entity relationships
   - Cross-entity rules

3. Model Layer: Entity validation
   - Attribute constraints
   - Entity-specific rules
   - State consistency

4. Persistence Layer: Database constraints
   - Unique constraints
   - Foreign key constraints
   - Data integrity
```

---

## 📝 Audit Trail

All entities maintain:
- Creation timestamp
- Last update timestamp
- Entity UUID for tracking
- Future: user_id for modifications