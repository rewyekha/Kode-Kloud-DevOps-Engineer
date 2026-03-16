# Weight: 19

## Fix Typo in Repository and Commit Changes

### Question

One of the Nautilus developers added some data under the repository:

```
/usr/src/kodekloudrepos/beta-t2q6
```

Later they realized that there was a **typo in one of the files**.

Inside the file **`lion-and-mouse.txt`**, the word **LION** was incorrectly spelled as **LIOON**.

#### Task

1. Fix the typo in the file.
2. Commit the changes with the message:

```
Fix typo in story title
```

Use the following credentials to access the storage server:

| Field    | Value     |
| -------- | --------- |
| Username | sarah     |
| Password | S3cure321 |

***

## Solution

We will access the repository, correct the typo, and commit the changes using Git.

***

## Step 1: Navigate to Sarah's Home Directory

```bash
cd ~
```

#### Terminal Output

```
[sarah@ststor01 /]$ cd ~
[sarah@ststor01 ~]$
```

***

## Step 2: Navigate to the Repository

```bash
cd /usr/src/kodekloudrepos/beta-t2q6
```

Verify the repository files.

```bash
ls
```

#### Terminal Output

```
[sarah@ststor01 ~]$ cd /usr/src/kodekloudrepos/beta-t2q6
[sarah@ststor01 beta-t2q6]$ ls
lion-and-mouse.txt
```

***

## Step 3: Edit the File

Open the file using `vi` and correct the typo.

```bash
vi lion-and-mouse.txt
```

After editing, verify the file content.

```bash
cat lion-and-mouse.txt
```

#### Terminal Output

```
[sarah@ststor01 beta-t2q6]$ cat lion-and-mouse.txt
--------------------------------------------
      THE LION AND THE MOUSE
--------------------------------------------

A Lion lay asleep in the forest, his great head resting on his paws.

A timid little Mouse came upon him unexpectedly, and in her fright and haste to get away, ran across the Lion's nose.

Roused from his nap, the Lion laid his huge paw angrily on the tiny creature to kill her.

"Spare me!" begged the poor Mouse. "Please let me go and some day I will surely repay you."

The Lion was much amused to think that a Mouse could ever help him. But he was generous and finally let the Mouse go.

Some days later, while stalking his prey in the forest, the Lion was caught in the toils of a hunter's net.

Unable to free himself, he filled the forest with his angry roaring.

The Mouse knew the voice and quickly found the Lion struggling in the net.

Running to one of the great ropes that bound him, she gnawed it until it parted, and soon the Lion was free.

"You laughed when I said I would repay you," said the Mouse. "Now you see that even a Mouse can help a Lion."
```

The typo **LIOON** has now been corrected to **LION**.

***

## Step 4: Check Git Status

```bash
git status
```

Example output:

```
On branch master
Changes not staged for commit:
        modified: lion-and-mouse.txt
```

***

## Step 5: Stage the Changes

```bash
git add lion-and-mouse.txt
```

***

## Step 6: Commit the Changes

```bash
git commit -m "Fix typo in story title"
```

#### Example Output

```
[master 3f21ab4] Fix typo in story title
 1 file changed, 1 insertion(+), 1 deletion(-)
```

***

## Step 7: Verify Repository Status

```bash
git status
```

Expected output:

```
On branch master
nothing to commit, working tree clean
```

***

## Final Result

* The typo **LIOON → LION** has been fixed.
* Changes have been committed successfully.
* Commit message used:

```
Fix typo in story title
```

✅ The repository update is completed successfully.
