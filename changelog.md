2025-03-12 implement update endpoint (#3) by @RafaelaRomin
[implement update endpoint]|

2025-03-12 Fix/controller path (#2) by @RafaelaRomin
[update route]|
[Merge branch 'master' into fix/controller-path]|

<<<<<<< HEAD
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
=======
2025-03-12 adjusting donations route (#1) by @RafaelaRomin
[adjusting donations route]|
>>>>>>> e16ce4b04651ad935ee3dcae3aebe546960a3c27
