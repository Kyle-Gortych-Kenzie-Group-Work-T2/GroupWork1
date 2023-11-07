<div align="right">
 
![week 2](https://img.shields.io/github/actions/workflow/status/Kyle-Gortych-Kenzie-Group-Work-T2/Week2GroupWork/main.yml?label=main) ![week 2](https://img.shields.io/github/actions/workflow/status/Kyle-Gortych-Kenzie-Group-Work-T2/Week2GroupWork/original.yml?label=original)

</div>

# Week 2 Group Project

descript

## Changes
<details>
<summary>Diff with original branch</summary>

<details>
<summary>SubscriptionFileStorage.java</summary>
 
```diff
diff --git a/GroupWork/SubscribeAndSave/src/main/java/com/kenzie/subscribeandsave/dao/SubscriptionFileStorage.java b/GroupWork/SubscribeAndSave/src/main/java/com/kenzie/subscribeandsave/dao/SubscriptionFileStorage.java
index 60b8821..282484a 100644
--- a/GroupWork/SubscribeAndSave/src/main/java/com/kenzie/subscribeandsave/dao/SubscriptionFileStorage.java
+++ b/GroupWork/SubscribeAndSave/src/main/java/com/kenzie/subscribeandsave/dao/SubscriptionFileStorage.java
@@ -49,8 +49,8 @@ public class SubscriptionFileStorage {
 
         StringBuilder sb = new StringBuilder();
         sb.append(subscription.getId()).append(",");
+        sb.append(subscription.getAsin()).append(","); //Swap getCustomerId and getAsin
         sb.append(subscription.getCustomerId()).append(",");
-        sb.append(subscription.getAsin()).append(",");
         sb.append(subscription.getFrequency());
         sb.append("\n");
 
@@ -82,9 +82,9 @@ public class SubscriptionFileStorage {
             if (subscriptionId.equals(id)) {
                 Subscription subscription = Subscription.builder()
                         .withSubscriptionId(id)
-                        .withAsin(subscriptionData[1])
-                        .withCustomerId(subscriptionData[2])
-                        .withFrequency(Integer.parseInt(subscriptionData[3]))
+                        .withCustomerId(subscriptionData[1].trim()) // Fix the order of CustomerId and Asin
+                        .withAsin(subscriptionData[2].trim()) // Fix the order of CustomerId and Asin
+                        .withFrequency(Integer.parseInt(subscriptionData[3].trim()))
                         .build();
                 return subscription;
             }
```
</details>

<details>
<summary>SubscriptionService.java</summary>
 
```diff
diff --git a/GroupWork/SubscribeAndSave/src/main/java/com/kenzie/subscribeandsave/service/SubscriptionService.java b/GroupWork/SubscribeAndSave/src/main/java/com/kenzie/subscribeandsave/service/SubscriptionService.java
index d7a4d0f..87b65a1 100644
--- a/GroupWork/SubscribeAndSave/src/main/java/com/kenzie/subscribeandsave/service/SubscriptionService.java
+++ b/GroupWork/SubscribeAndSave/src/main/java/com/kenzie/subscribeandsave/service/SubscriptionService.java
@@ -48,34 +48,50 @@ public class SubscriptionService {
     public Subscription subscribe(String customerId, String asin, int frequency) {
         if (StringUtils.isBlank(customerId) || StringUtils.isBlank(asin)) {
             throw new IllegalArgumentException(
-                String.format(
-                    "Invalid inputs. A Customer ID and ASIN must be provided. Provided: {Customer ID: %s, ASIN: %s}",
-                    customerId,
-                    asin)
+                    String.format(
+                            "Invalid inputs. A Customer ID and ASIN must be provided. Provided: {Customer ID: %s, ASIN: %s}",
+                            customerId,
+                            asin)
             );
         }
+        // TODO BUG 2 FIX - CHECK FOR NON-SUBSCRIBABLE ASINS (isSNS=false in catalog.json), THROW EXCEPTION
+
+        /*
+        if (asin.equals("B072PR8QNN") || asin.equals("B07R5QD598") || asin.equals("B079BG3LQF") ) {
+
+            throw new IllegalArgumentException(
+                    String.format(
+                            "Invalid asin value. Is not a subscribable product " +
+                                    " Provided: {asin: %s}",
+                            asin));
+
+        }
+         */
 
         if (frequency < 1 || frequency > 6) {
             throw new IllegalArgumentException(
-                String.format(
-                    "Invalid frequency value. Please provide how often (in months) the " +
-                        "subscription should occur - between 1 and 6. Provided: {Frequency: %d}",
-                    frequency));
+                    String.format(
+                            "Invalid frequency value. Please provide how often (in months) the " +
+                                    "subscription should occur - between 1 and 6. Provided: {Frequency: %d}",
+                            frequency));
         }
 
         if (!identityService.validateCustomer(customerId)) {
             throw new IllegalArgumentException(
-                String.format("Unable to create subscription for customerId: %s. Unknown customer.", customerId)
+                    String.format("Unable to create subscription for customerId: %s. Unknown customer.", customerId)
             );
         }
 
+        // TODO FIX BUG 1 - UNKNOWN ASIN - FIX STRING ERROR MESSAGE
         Product product = productService.getProductByAsin(asin);
         if (product == null) {
             throw new IllegalArgumentException(
-                String.format("Unable to create subscription for ASIN: % s. Unrecognized ASIN.", asin)
+                    String.format("Unable to create subscription for ASIN: %s Unrecognized ASIN.", asin)
             );
         }
-
+        if(!product.isSNS()) {
+            throw new IllegalArgumentException("product isn't valid");
+        }
         return subscriptionDAO.createSubscription(customerId, asin, frequency);
     }
```
</details>

<details>
<summary>bug_test_case.md 1, 2, and 3</summary>
 
```diff
diff --git a/GroupWork/SubscribeAndSave/src/test/java/com/kenzie/subscribeandsave/bug_1_test_case.md b/GroupWork/SubscribeAndSave/src/test/java/com/kenzie/subscribeandsave/bug_1_test_case.md
new file mode 100644
index 0000000..e4499c4
--- /dev/null
+++ b/GroupWork/SubscribeAndSave/src/test/java/com/kenzie/subscribeandsave/bug_1_test_case.md
@@ -0,0 +1,11 @@
+Class: <class with bug>
+
+**[methodUnderTest] _ [testCondition] _ [expectedBehavior]**
+d* **Description**: [short description of the test case]
+* GIVEN
+    * [bulleted list of relevant pre-conditions for the test to run (usually data you're setting up to test)]
+* WHEN
+    1. [ordered list of methods you will call with a description of relevant arguments]
+    2. [most of your test cases will have a single WHEN item, but if you want more than one keep this line]
+* THEN
+    * [bulleted list of verifications that you will perform to see if the test case passes]
\ No newline at end of file
diff --git a/GroupWork/SubscribeAndSave/src/test/java/com/kenzie/subscribeandsave/bug_2_test_case.md b/GroupWork/SubscribeAndSave/src/test/java/com/kenzie/subscribeandsave/bug_2_test_case.md
new file mode 100644
index 0000000..9897f6b
--- /dev/null
+++ b/GroupWork/SubscribeAndSave/src/test/java/com/kenzie/subscribeandsave/bug_2_test_case.md
@@ -0,0 +1,11 @@
+Class: <class with bug>
+
+**[methodUnderTest] _ [testCondition] _ [expectedBehavior]**
+* **Description**: [short description of the test case]
+* GIVEN
+    * [bulleted list of relevant pre-conditions for the test to run (usually data you're setting up to test)]
+* WHEN
+    1. [ordered list of methods you will call with a description of relevant arguments]
+    2. [most of your test cases will have a single WHEN item, but if you want more than one keep this line]
+* THEN
+    * [bulleted list of verifications that you will perform to see if the test case passes]
\ No newline at end of file
diff --git a/GroupWork/SubscribeAndSave/src/test/java/com/kenzie/subscribeandsave/bug_3_test_case.md b/GroupWork/SubscribeAndSave/src/test/java/com/kenzie/subscribeandsave/bug_3_test_case.md
new file mode 100644
index 0000000..084cae5
--- /dev/null
+++ b/GroupWork/SubscribeAndSave/src/test/java/com/kenzie/subscribeandsave/bug_3_test_case.md
@@ -0,0 +1,10 @@
+Class: SubscriptionService
+**[methodUnderTest] _ [testCondition] _ [expectedBehavior]**
+* **Description**: [short description of the test case]
+* GIVEN
+    * [bulleted list of relevant pre-conditions for the test to run (usually data you're setting up to test)]
+* WHEN
+    1. [ordered list of methods you will call with a description of relevant arguments]
+    2. [most of your test cases will have a single WHEN item, but if you want more than one keep this line]
+* THEN
+    * [bulleted list of verifications that you will perform to see if the test case 
\ No newline at end of file
```
</details>

<details>
<summary>SubscriptionServiceTest.java</summary>
 
```diff
diff --git a/GroupWork/SubscribeAndSave/src/test/java/com/kenzie/subscribeandsave/service/SubscriptionServiceTest.java b/GroupWork/SubscribeAndSave/src/test/java/com/kenzie/subscribeandsave/service/SubscriptionServiceTest.java
index 6423974..14dbcfb 100644
--- a/GroupWork/SubscribeAndSave/src/test/java/com/kenzie/subscribeandsave/service/SubscriptionServiceTest.java
+++ b/GroupWork/SubscribeAndSave/src/test/java/com/kenzie/subscribeandsave/service/SubscriptionServiceTest.java
@@ -1,6 +1,7 @@
 package com.kenzie.subscribeandsave.service;
 
 import com.kenzie.subscribeandsave.App;
+import com.kenzie.subscribeandsave.dao.SubscriptionFileStorage;
 import com.kenzie.subscribeandsave.types.Subscription;
 import com.kenzie.subscribeandsave.util.SubscriptionRestorer;
 import org.apache.commons.lang.StringUtils;
@@ -8,7 +9,9 @@ import org.junit.jupiter.api.AfterEach;
 import org.junit.jupiter.api.BeforeEach;
 import org.junit.jupiter.api.Test;
 
-import static org.junit.jupiter.api.Assertions.fail;
+import java.io.File;
+
+import static org.junit.jupiter.api.Assertions.*;
 
 public class SubscriptionServiceTest {
     private static final String ASIN = "B01BMDAVIY";
@@ -35,15 +38,15 @@ public class SubscriptionServiceTest {
     @Test
     public void runAllTests() {
         classUnderTest = new SubscriptionService(App.getAmazonIdentityService(), App.getSubscriptionDAO(),
-                                                 App.getAmazonProductService());
+                App.getAmazonProductService());
         boolean pass = true;
 
         pass = subscribe_newSubscription_subscriptionReturned();
         pass = getSubscription_existingSubscription_subscriptionReturned() && pass;
         pass = subscribe_unknownCustomer_exceptionOccurs() && pass;
-        pass = test_bug1() && pass;
-        pass = test_bug2() && pass;
-        pass = test_bug3() && pass;
+        pass = subscribe_invalidCustomerIdUnknownASIN_throwsIlligalArguementException() && pass;
+        pass = subscribe_nonSubscribableItem_IlligalArguementException() && pass;
+        pass = getSubscriptionById_correctOrderOfValues_subscriptionReturned() && pass;
 
         if (!pass) {
             String errorMessage = "\n/!\\ /!\\ /!\\ The SubscriptionService tests failed. Test aborted. /!\\ /!\\ /!\\";
@@ -68,7 +71,7 @@ public class SubscriptionServiceTest {
         }
         if (!subscriptionId.equals(result.getId())) {
             System.out.println("   FAIL: Subscription returned when getting subscription by id has mismatching id " +
-                                   "value");
+                    "value");
             return false;
         }
 
@@ -93,7 +96,7 @@ public class SubscriptionServiceTest {
         }
         if (StringUtils.isBlank(result.getId())) {
             System.out.println("   FAIL: Creating subscription should return a subscription with a populated id " +
-                                   "field.");
+                    "field.");
             return false;
         }
 
@@ -120,39 +123,68 @@ public class SubscriptionServiceTest {
     }
 
     // PARTICIPANTS: Fill in the example test below after fixing Bug 1 - refactor as needed
-    public boolean test_bug1() {
-        // GIVEN
 
+    public boolean subscribe_invalidCustomerIdUnknownASIN_throwsIlligalArguementException() {
+        // GIVEN
+        String customerId = CUSTOMER_ID;
+        String asin = "12532423";
+        int frequency = 1;
         // WHEN
+        try {
+            Subscription result = classUnderTest.subscribe(customerId, asin, frequency);
+        } catch (IllegalArgumentException w) {
+            System.out.println("  PASS: Cannot subscribe with invalid asin.");
+            return true;
+        }
+
 
         // THEN
 
-        System.out.println("   FAIL: Need to implement test to fix Bug 1!");
+        System.out.println("  FAIL: An exception should have occurred when subscribing invalid asin.");
         return false;
     }
 
     // PARTICIPANTS: Rename and fill in the example test below after fixing Bug 2 - refactor as needed
-    public boolean test_bug2() {
+
+    public boolean subscribe_nonSubscribableItem_IlligalArguementException() {
         // GIVEN
+        String customerId = CUSTOMER_ID;
+        String asin= "B07R5QD598";
+        int frequency = 1;
 
         // WHEN
-
-        // THEN
-
-        System.out.println("   FAIL: Need to implement test to fix Bug 2!");
+        try{
+            Subscription result = classUnderTest.subscribe(customerId,asin,frequency);
+        }catch (IllegalArgumentException w){
+            System.out.println("PASS:Cannot subscribe with invalid ASIN");
+            return true;
+        }
+        System.out.println("FAIL:Need to implement test to fix Bug 2!");
         return false;
     }
 
+
     // PARTICIPANTS: Rename and fill in the example test below after fixing Bug 3 - refactor as needed
-    public boolean test_bug3() {
+
+    public boolean getSubscriptionById_correctOrderOfValues_subscriptionReturned() {
         // GIVEN
+        String customerId = "amzn1.account.AEZI3A027560538W420H09ACTDP2";
+        String asin = "B00006IEJB";
+        int frequency = 3;
 
-        // WHEN
+        //SubscriptionFileStorage storage = new SubscriptionFileStorage(new File("/home/kg/main/School/kenzie_academy/SE_Backend/SE_semester_2/repos/ata-week-2-kylegortych/GroupWork/SubscribeAndSave/src/main/resources/subscriptions.csv"));//Change directary to your local michines directory
 
-        // THEN
+        // WHEN - Retrieve the subscription by ID
+        Subscription result = classUnderTest.getSubscription("81a9792e-9b4c-4090-aac8-28e733ac2f54");
 
-        System.out.println("   FAIL: Need to implement test to fix Bug 3!");
-        return false;
+        // THEN - Verify that the subscription was retrieved correctly with the correct order of values
+        //assertNotNull(result, "Expected a subscription to be retrieved.");
+        if(!customerId.equals(result.getCustomerId()) && asin.equals(result.getAsin())){ return false;}
+        //assertEquals(asin, result.getAsin(), "ASIN should match.");
+        //assertEquals(frequency, result.getFrequency(), "Frequency should match.");
+
+        System.out.println("  PASS: Test for correct order of values in getSubscriptionById succeeded.");
+        return true;
     }
 
 
@@ -161,4 +193,4 @@ public class SubscriptionServiceTest {
     public void restoreSubscriptions() {
         SubscriptionRestorer.restoreSubscriptions();
     }
-}
+}
\ No newline at end of file
```
</details>

</details>

<div align="center">
 
### :hammer_and_wrench: Tools :

| Build System | Languages |
| ------------ | --------- |
| <img src="https://img.shields.io/badge/Gradle-white?style=plastic&logo=gradle&logoColor=black" title="gradle" alt="gradle" height="30"/> | <img src="https://custom-icon-badges.demolab.com/badge/Java-white.svg?&sytle=plastic&logo=java" title="Java" alt="Java" height="30"/> |
</div>
<br>

### Gradle commands

```console
./gradlew groupwork-subscribeandsave-test
```
<br>

<a href="your-gmail-link?">:mailbox:</a> How to reach the maintainer
