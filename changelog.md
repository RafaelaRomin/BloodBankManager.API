# 🆕 Changelog - 2025-03-12

## ✅ Pull Requests Merged

### 🔹 implement update endpoint (#3) by @RafaelaRomin
  **Commits:**
  - **implement update endpoint** (by @Rafaela Romin)

  📝 **BloodBankManager.API/Controllers/DonationController.cs**
  ```diff
  @@ -31,11 +31,11 @@ public async Task<IActionResult> GetAll()
         [HttpGet("last-30-days")]
         public async Task<IActionResult> GetLast30Days()
         {
-           var query = new GetDonationsRelatoryQuery();
-           
-           var donationsLast30Days = await _mediator.Send(query);
+            var query = new GetDonationsRelatoryQuery();
 
-           return Ok(donationsLast30Days);
+            var donationsLast30Days = await _mediator.Send(query);
+
+            return Ok(donationsLast30Days);
         }
 
         [HttpGet("{id}")]
@@ -55,5 +55,14 @@ public async Task<IActionResult> Post(CreateDonationCommand createDonationComman
 
             return CreatedAtAction(nameof(GetById), new { id }, createDonationCommand);
         }
+
+
+        [HttpPut]
+        public async Task<IActionResult> Update(CreateDonationCommand createDonationCommand)
+        {
+            var id = await _mediator.Send(createDonationCommand);
+
+            return CreatedAtAction(nameof(GetById), new { id }, createDonationCommand);
+        }
     }
 }
  ```

### 🔹 Fix/controller path (#2) by @RafaelaRomin
  **Commits:**
  - **update route** (by @Rafaela Romin)

  📝 **.github/workflows/changelog.yaml**
  ```diff
  @@ -36,14 +36,26 @@ jobs:
               .filter(pr => pr.merged_at && new Date(pr.merged_at) >= since)
               .map(pr => `- ${pr.title} (#${pr.number}) by @${pr.user.login}`);
 
-            const changelogContent = `# 🆕 Changelog - ${new Date().toISOString().split('T')[0]}\n\n` +
-                                    `## ✅ Pull Requests Merged \n` +
-                                    (mergedPRs.length ? mergedPRs.join("\n") : "No updates this week.");
+            require("fs").writeFileSync("pr_changelog.md", mergedPRs.length ? mergedPRs.join("\n") : "No updates this week.");
 
-            require("fs").writeFileSync("changelog.md", changelogContent);
+      - name: Get Commits from last week
+        run: |
+          echo "# 🆕 Changelog - $(date +'%d/%m/%Y')" > changelog.md
+          echo "" >> changelog.md
+
+          # Obter commits dos últimos 7 dias
+          commits=$(git log --pretty=format:"- %s (%h) by %an" --since="1 week ago")
+
+          echo "## 🛠 Alterações" >> changelog.md
+          echo "$commits" | grep -iE "(update|change|modify|refactor|🔄)" || echo "Nenhuma alteração relevante." >> changelog.md
+          echo "" >> changelog.md
+
+          # Adicionar PRs ao changelog
+          echo "## ✅ Pull Requests Merged" >> changelog.md
+          cat pr_changelog.md >> changelog.md
+          echo "" >> changelog.md
 
-      - name: Exibir Changelog
-        run: cat changelog.md
+          cat changelog.md
 
       - name: Commit and Changelog Push
         run: |
  ```

  📝 **BloodBankManager.API/Controllers/DonorController.cs**
  ```diff
  @@ -60,7 +60,7 @@ public async Task<IActionResult> GetById(int id)
             return Ok(donor);
         }
 
-        [HttpGet("{id}/donations")]
+        [HttpGet("{id}/all-donations")]
         public async Task<IActionResult> GetDonationsOfDonor(int id)
         {
             var query = new GetAllDonationsByDonorIdQuery(id);
  ```

  📝 **BloodBankManager.Application/Commands/CreateDonation/CreateDonationCommandHandler.cs**
  ```diff
  @@ -2,11 +2,6 @@
 using BloodBankManager.Core.Repositories;
 using BloodBankManager.Core.Services;
 using MediatR;
-using System;
-using System.Collections.Generic;
-using System.Linq;
-using System.Text;
-using System.Threading.Tasks;
 
 namespace BloodBankManager.Application.Commands.CreateDonation
 {
  ```

  📝 **BloodBankManager.Application/Services/BloodStorageService.cs**
  ```diff
  @@ -19,13 +19,11 @@ public async Task AddBloodStorage(int donorId, int quantityMl)
             var donor = await _donorRepository.GetDonorByIdAsync(donorId);
 
             var storage = await _bloodStorageRepository
-                                .GetBloodTypeAndRhFactor(donor.BloodType, donor.RhFactor);
+                .GetBloodTypeAndRhFactor(donor.BloodType, donor.RhFactor);
 
-           storage.UpdateQuantity(quantityMl);
+            storage.UpdateQuantity(quantityMl);
 
             _bloodStorageRepository.UpdateStorage(storage);
-
-
         }
     }
 }
  ```
  - **Merge branch 'master' into fix/controller-path** (by @Rafaela Romin)

  📝 **.github/workflows/changelog.yaml**
  ```diff
  @@ -2,15 +2,25 @@ name: Generate Changelog
 
 on:
   schedule:
-    - cron: '0 12 * * 1' #toda segunda-feira 12h
+    - cron: '0 12 * * 1' # Roda toda segunda-feira 12h
   workflow_dispatch: # Permite rodar manualmente 
 
+permissions:
+  contents: write 
+
 jobs:
   generate-changelog:
     runs-on: ubuntu-latest
     steps:
       - name: Checkout code
-        uses: actions/checkout@v2
+        uses: actions/checkout@v3
+        with:
+          persist-credentials: false
+
+      - name: Configure Git User
+        run: |
+          git config --global user.name "github-actions[bot]"
+          git config --global user.email "github-actions[bot]@users.noreply.github.com"
 
       - name: Get All Pull Requests merged
         uses: actions/github-script@v6
@@ -19,7 +29,7 @@ jobs:
           script: |
             const { owner, repo } = context.repo;
             const since = new Date();
-            since.setDate(since.getDate() - 7); // Últimos 7 dias
+            since.setDate(since.getDate() - 7); //Last 7 days
 
             const pulls = await github.paginate(
               github.rest.pulls.list,
@@ -46,22 +56,52 @@ jobs:
           # Obter commits dos últimos 7 dias
           commits=$(git log --pretty=format:"- %s (%h) by %an" --since="1 week ago")
 
+          # Categorizar commits
+          echo "## 🔥 Novas Features" >> changelog.md
+          echo "$commits" | grep -iE "(add|new|feature|create|implement|🚀)" || echo "Nenhuma nova funcionalidade." >> changelog.md
+          echo "" >> changelog.md
+
           echo "## 🛠 Alterações" >> changelog.md
           echo "$commits" | grep -iE "(update|change|modify|refactor|🔄)" || echo "Nenhuma alteração relevante." >> changelog.md
           echo "" >> changelog.md
 
+          echo "## ⚠️ Quebras de Compatibilidade" >> changelog.md
+          echo "$commits" | grep -iE "(remove|delete|break|incompatible|❌)" || echo "Nenhuma quebra detectada." >> changelog.md
+          echo "" >> changelog.md
+
           # Adicionar PRs ao changelog
           echo "## ✅ Pull Requests Merged" >> changelog.md
           cat pr_changelog.md >> changelog.md
           echo "" >> changelog.md
 
           cat changelog.md
+            require("fs").writeFileSync("pr_changelog.md", mergedPRs.length ? mergedPRs.join("\n") : "No updates this week.");
 
-      - name: Commit and Changelog Push
+      - name: Get Commits from last week
+        run: |
+          echo "# 🆕 Changelog - $(date +'%d/%m/%Y')" > changelog.md
+          echo "" >> changelog.md
+
+          # Obter commits dos últimos 7 dias
+          commits=$(git log --pretty=format:"- %s (%h) by %an" --since="1 week ago")
+
+          echo "## 🛠 Alterações" >> changelog.md
+          echo "$commits" | grep -iE "(update|change|modify|refactor|🔄)" || echo "Nenhuma alteração relevante." >> changelog.md
+          echo "" >> changelog.md
+
+          # Adicionar PRs ao changelog
+          echo "## ✅ Pull Requests Merged" >> changelog.md
+          cat pr_changelog.md >> changelog.md
+          echo "" >> changelog.md
+
+          cat changelog.md
+
+      - name: Commit and Push Changelog
         run: |
-          git config --global user.name "github-actions[bot]"
-          git config --global user.email "github-actions[bot]@users.noreply.github.com"
           git add changelog.md
           git commit -m "📝 Update Changelog"
-          git push
-        continue-on-error: true 
+          git branch -M master # Garante que está na branch correta
+          git remote set-url origin https://x-access-token:${{ secrets.GITHUB_TOKEN }}@github.com/${{ github.repository }}.git
+          git push origin master
+        env:
+          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
  ```

  📝 **changelog.md**
  ```diff
  @@ -0,0 +1,12 @@
+# 🆕 Changelog - 12/03/2025
+
+## 🔥 Novas Features
+Nenhuma nova funcionalidade.
+
+## 🛠 Alterações
+
+## ⚠️ Quebras de Compatibilidade
+Nenhuma quebra detectada.
+
+## ✅ Pull Requests Merged
+- adjusting donations route (#1) by @RafaelaRomin
  ```

### 🔹 adjusting donations route (#1) by @RafaelaRomin
  **Commits:**
  - **adjusting donations route** (by @Rafaela Romin)

  📝 **BloodBankManager.API/Controllers/DonorController.cs**
  ```diff
  @@ -60,7 +60,7 @@ public async Task<IActionResult> GetById(int id)
             return Ok(donor);
         }
 
-        [HttpGet("{id}/all-donations")]
+        [HttpGet("{id}/donations")]
         public async Task<IActionResult> GetDonationsOfDonor(int id)
         {
             var query = new GetAllDonationsByDonorIdQuery(id);
  ```
