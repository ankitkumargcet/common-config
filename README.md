# common-config
This repository will be used to share common properties and pom files for multiple microservices

**# GitHub Deployment Strategy with Release Branch Management**

## **Branching Strategy**
We follow the below GitHub branching strategy for structured deployments:

- **`main`** → Always reflects the latest stable production release.
- **`develop`** → Contains completed and tested features before creating a release.
- **`develop-bridge`** → Intermediate branch for integrating and testing multiple feature branches before merging into `develop`.
- **`feature/*`** → Used for developing new features.
- **`release/*`** → Used for preparing a versioned release.
- **`hotfix/*`** → For urgent fixes applied to production.
- **`bugfix/*`** → Used for fixing bugs identified before release.
- **`spike/*`** → Used for exploratory work, proof of concepts, or experimental development.
- **`feature/shared-*`** → Used when multiple developers work on shared functionality required by different feature branches.

---

## **Deployment Process**

### **1. Feature Development & Testing in E1**
1. Developers create a **feature branch** (`feature/X`).
2. If a feature requires shared functionality, it is developed in a **shared feature branch** (`feature/shared-X`).
3. Feature branches are deployed directly to **E1** via the PaaS template.
4. Once tested, the feature is promoted to **E2**.
5. After validation, the feature branch is merged into **`develop-bridge`**, not directly into `develop`.
6. **Multiple features are integrated and tested together in `develop-bridge`** before moving to `develop`.
7. Once all integrated features pass testing in `develop-bridge`, it is merged into `develop` to ensure a stable build.
   ```sh
   git checkout develop
   git merge --no-ff develop-bridge
   git push origin develop
   ```

### **2. Creating a Release & Deploying Sequentially (E1 → E2 → E3)**
1. A **release branch** is created from `develop` (e.g., `release/1.x.x`).
   ```sh
   git checkout develop
   git checkout -b release/1.x.x
   git push origin release/1.x.x
   ```
2. `release/1.x.x` is deployed to **E1 → E2 → E3** sequentially.
3. Once validated in E3, a PR is raised to merge `release/1.x.x` into `main`.
   ```sh
   git checkout main
   git merge --no-ff release/1.x.x
   git push origin main
   ```
4. A release tag is created:
   ```sh
   git tag -a v1.x.x -m "Release v1.x.x"
   git push origin v1.x.x
   ```
5. Finally, `release/1.x.x` is merged back into `develop` to sync production changes.
   ```sh
   git checkout develop
   git merge --no-ff release/1.x.x
   git push origin develop
   ```

---

## **Handling Hotfixes**
If an urgent production fix is required:
1. Create a **hotfix branch** from `release/1.x.x`.
   ```sh
   git checkout -b hotfix/1.x.x-fix release/1.x.x
   ```
2. Apply the fix and test in **E1 → E2 → E3**.
3. Merge `hotfix/1.x.x-fix` into `release/1.x.x`, `main`, and `develop`.
   ```sh
   git checkout release/1.x.x
   git merge --no-ff hotfix/1.x.x-fix
   git push origin release/1.x.x
   ```
   ```sh
   git checkout main
   git merge --no-ff hotfix/1.x.x-fix
   git tag -a v1.x.x+1 -m "Hotfix for v1.x.x"
   git push origin v1.x.x+1
   ```
   ```sh
   git checkout develop
   git merge --no-ff hotfix/1.x.x-fix
   git push origin develop
   ```

---

## **Key Rules to Follow**
- **Feature branches (`feature/*`) must be merged into `develop-bridge`, not directly into `develop`.**
- **`develop-bridge` serves as an intermediate branch to integrate and test multiple feature branches.**
- **Once tested, `develop-bridge` is merged into `develop` as a stable version.**
- **Only release branches (`release/*`) are deployed sequentially from E1 → E2 → E3.**
- **`release/*` must be merged into `main`, NOT `develop`.**
- **After merging into `main`, `release/*` is merged back into `develop`.**
- **`hotfix/*` branches are merged into `release/*`, `main`, and `develop`.**
- **Release branches remain active for patches and future fixes.**
- **Use `feature/shared-*` for shared functionality across multiple feature branches.**
- **`spike/*` is used only for research or experimental changes and should not be merged directly into `develop`.**
- **`bugfix/*` branches are used to fix critical issues before release deployment.**

This strategy ensures a **structured, stable, and rollback-safe** deployment process. 🚀
