# Hash investigation writeup

1. Explaining your reasoning, identify which cryptographic hashes were used to produce:

   1. *hashset-earlier* = **SHA1**
      - `> head -n 1 hashset-earlier | cut -d' ' -f1 | wc -c`
      - then -1 from count given (exclude newline) and use Cyber Chef to put data through each hashing algorithm and get character count
      - 40 hex characters 
   3. *hashset-bad* = **MD5**
      - `> head -n 1 hashset-good | wc -c`
      - then -1 from count given (exclude newline) and use Cyber Chef to put data through each hashing algorithm and get character count
      - 32 hex characters 
   4. *hashset-good* = **SHA256**
      - `> head -n 1 hashset-good | wc -c`
      - then -1 from count given (exclude newline) and use Cyber Chef to put data through each hashing algorithm and get character count
      - 64 hex characters 

2. Identify which files are present in the archive but have different content from when *hashset-earlier* was taken.
   - `sha1sum -c ../hashset-earlier | grep -e "FAILED$"`
   - files:
     - **f0022**, **f0148**, **f0254**, **f0261**, **f0407**, **f0478**

3. Identify which files have been deleted from the archive that were present when the *hashset-earlier* was taken.
   - `sha1sum -c ../hashset-earlier | grep -e "FAILED open or read"`
   - files:
     - **f0128**, **f0258**, **f0480**, **f0490**

4. Identify which files in *hashset-earlier* are duplicate copies of other files in *hashset-earlier*. Ensure your answer unambiguously groups together the files that are duplicates of each other.
   - `cat  hashset-earlier | sort | uniq -w 40 -D`
   - files:
     - **f0063** and **f0462**
     - **f0098**, **f0127**, and **f0153**
     - **f0155**, **f0288**, and **f0458**
     - **f0081** and **f0161**
     - **f0064** and **f0144**
     - **f0315** and **f0471**

5. Identify which files in the archive have been renamed from the name used in *hashset-earlier* to the name used in the archive. Ensure your answer is unambiguous as to which name is "from" and which name is "to".
   - a) redo hashes of files: `for i in $(ls -f); do sha1sum  $i >> newhash ; done && sort newhash > new-hashes`
   - b) compare files to what hashes are duplicates on first 32 characters (hashes), but different overall: `sort new-hashes earlier-hashes | uniq -w 32 -D | sort | uniq -u`
   - c) for files found, cross reference to see which files were present *hashset-earlier* was taken but have different content (question 2), and what files are no longer here but were here when *hashset-earlier* was taken (question 3)
   - files: 
     - * from: **f0480** -- to: **f0478**


6. Identify which files in the archive are known malware.
    - a) create md5 checksums of files in archive and sort: `for i in $(ls -f); do  md5sum  $i >> md5s ; done && sort md5s > md5-hashset`
    - b) create a file with the known duplicate files 
    - c) compare the files (excluding known duplicates): `sort ../hashset-bad md5-hashset | uniq -w 32 -D | grep -vf duplicates.txt`
    - files (14):
      - **f0233**, **f0191**, **f0482**, **f0398**, **f0147**, **f0135**, **f0097**, **f0462**, **f0012**, **f0328**, **f0035**, **f0353**, **f0496**, **f0239**

7. Identify which files in the archive are neither known good system configuration files, nor known malware.
   - a) get files that are not in *hashset-bad* (using previously made md5-hashset): `sort hashset-bad md5-hashset | uniq -w 32 -u | grep -E " original/*" > notbad`
   - b) make sha256 hashes of files `for i in $(ls -f); do  sha256sum  $i >> sha256s ; done && sort sha256s > sha256-hashset`
   - c) get files that are not in *hashset-good*: `sort hashset-good sha256-hashset | uniq -w 64 -u | grep -E " original/*" > notgood`
   - d) get md5sum hashes from all the files in notgood: `for i in $(cat notgood | grep -Eo " original/.*"); do  md5sum $i >> notgoodMD5s ; done && sort notgoodMD5s > notgoodMD5-hashset`
   - e) compare hashes in *notgoodMD5s* and *notbad* for duplicates (files not in good and not in bad): `sort notbad notgoodMD5-hashset | uniq -w 32 -d`
   - files (17):
     - **f0407**, **f0410**, **f0261**, **f0171**, **f0022**, **f0467**, **f0119**, **f0142**, **f0096**, **f0148**, **f0493**, **f0394**, **f0362**, **f0050**, **f0254**, **f0213**, **f0489**

