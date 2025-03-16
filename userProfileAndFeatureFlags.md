# UserProfile & Feature Flags Implementation Guide

## 1. UserProfile Implementation

### **File**: `app/common/services/profile/profile-storage.js`
### **Class**: `UserProfile`

#### **Methods**:
| Method | Purpose | Example |
|--------|---------|---------|
| `constructor()` | Initializes `cachedUserProfile` and `featureFlags` | `this.cachedUserProfile = null` |
| `setProfile(profileData)` | Saves profile to `localStorage` and caches in memory | Called after API fetch in sagas |
| `getProfile()` | Returns cached profile data | `const userId = UserProfile.getProfile().userId` |
| `getSessionPermission(flag)` | Checks permissions from profile (e.g., `pilotProperties`) | Used in `checkUserProfilePermission()` |

---

### **Flow**:
1. **Initialization**  
   - Triggered by landing page `saga.js` on app load.  
   - Calls API to fetch user data → stores via `UserProfile.setProfile(response)`.  

2. **Data Storage**  
   - Profile persisted in `localStorage` under key `userprofile`.  
   - In-memory cache (`cachedUserProfile`) avoids frequent `localStorage` reads.  

3. **Usage in Components**  
   ```javascript
   // Get user metadata
   const { firstName, departmentId } = UserProfile.getProfile();

   // Check permission
   const canEdit = UserProfile.getSessionPermission("EDIT_PERM") === "Y";
   ```

---

## 2. Feature Flags Implementation

### **Files**:
- `sagajs` (API calls)  
- `app/common/helpers/flagUtils.js` (flag checks)  
- `app/common/helpers/toggleFeatureFlagUtil.js` (flag updates)  

---

### **Key Methods**:
| File | Method | Purpose |  
|------|--------|---------|  
| `sagajs` | `initfetchFeatureFlagAPIRequest()` | Fetches flags from API |  
| `toggleFeatureFlagUtil.js` | `populateFeatureEnablementFlagsOnload()` | Merges API flags with `localStorage.featureEnablement` |  
| `flagUtils.js` | `checkFeatureFlagStatus(flag)` | Checks if a flag is enabled |  
| `flagUtils.js` | `checkUserProfilePermission(flag)` | Validates permissions against profile |  

---

### **Flow**:
1. **Fetch Flags**  
   - Saga `fetchFeatureFlags` calls API → response stored via `UserProfile.setFeatureFlags()`.  
2. **LocalStorage Sync**  
   - `populateFeatureEnablementFlagsOnload()` merges API flags with existing `localStorage.featureEnablement`.  
3. **UI Integration**  
   ```javascript
   // Component Example
   import { checkFeatureFlagStatus } from 'flagUtils';

   if (checkFeatureFlagStatus("NEW_CHECKOUT")) {
     renderNewCheckout();
   }
   ```

4. **Permission Gating**  
   ```javascript
   // Restrict access to admins
   if (checkUserProfilePermission("ADMIN_DASHBOARD")) {
     showAdminPanel();
   }
   ```

---

## 3. Critical Integration

### **Where They Intersect**:
- **Flag-Driven Permissions**: Combine `checkFeatureFlagStatus()` + `checkUserProfilePermission()`.  
   ```javascript
   if (
     checkFeatureFlagStatus("BETA_FEATURE") &&
     checkUserProfilePermission("BETA_ACCESS")
   ) {
     enableBetaFeature();
   }
   ```
---

## 4. Development Examples

### **Adding a New Feature Flag**:
1. **Backend**: Add flag to `featureFlagDetails.profileDetails` API response.  
2. **Frontend**:  
   ```javascript
   // In React Component
   if (checkFeatureFlagStatus("YOUR_FLAG")) {
     return <NewFeatureComponent />;
   }
   ```

### **Adding a Permission**:
1. **Backend**: Include key in user profile (e.g., `permissions.MY_PERM: "Y"`).  
2. **Frontend**:  
   ```javascript
   if (checkUserProfilePermission("MY_PERM")) {
     grantAccess();
   }
   ```
---

## 5. Folder Structure
```
app/  
├── common/  
│   ├── services/  
│   │   └── profile/  
│   │       └── profile-storage.js (UserProfile class)  
│   └── helpers/  
│       ├── flagUtils.js (flag checks)  
│       └── toggleFeatureFlagUtil.js (flag updates)  
└── landing-page/  
    └── sagajs (API calls)  
```
