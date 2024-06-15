# bookshelf
*Microsoft Access database for keeping track of my book hauls.*

List of queries:

1. <details><summary> <b>Author/Book Lookup</b>: Identifies specific records based on either author's name or book title.</summary>

   ```sql
   SELECT *
   FROM Books
   WHERE Author LIKE "*" & [Enter Author Name (Leave blank for all):] & "*" 
     AND [Book Title] LIKE "*" & [Enter Book Title (Leave blank for all):] & "*";

</details>

2. <details><summary><b>Finished Books</b>: Tracks completed reads.</summary>

    ```sql
   SELECT Author, [Book Title]
   FROM Books
   WHERE Completed = TRUE
   ORDER BY Author;

</details>

3. <details><summary><b>Incomplete Series - Unreleased Titles from Purchased Series</b>: Displays all unreleased titles from already purchased series listed by estimated release date.</summary>

    ```sql
    SELECT Author, [Book Title], Sequence, [Estimated Release Date]
    FROM Books AS b
    WHERE [Book Series] IN (
        SELECT [Book Series]
        FROM Books
        WHERE [Purchased?] LIKE "Yes"
        AND [Book Series] <> "Standalone"
    )
    AND [Released?] = False
    AND [Book Series] <> "Standalone"
    ORDER BY IIf(IsDate([Estimated Release Date]), CDate([Estimated Release Date]), DateSerial(9999,12,31)), Author;


</details>

4. <details><summary><b>List of Authors - Progress Tracking</b>: Tracks totals for read versus unread books for each author.</summary>

    ```sql
   SELECT Author, SUM(IIF([Completed] = True, 1, 0)) AS [Total Read], SUM(IIF([Completed] = False, 1, 0)) AS [Total Unread]
   FROM Books
   GROUP BY Author
   ORDER BY Author;

</details>

5. <details><summary><b>List of Authors - Purchases</b>: Calculates total number of purchased books per author.</summary>

    ```sql
    SELECT Author, COUNT(IIF([Purchased?] = 'Yes', 1, NULL)) AS [Owned Books]
    FROM Books
    GROUP BY Author
    HAVING COUNT(IIF([Purchased?] = 'Yes', 1, NULL)) > 0
    ORDER BY COUNT(IIF([Purchased?] = 'Yes', 1, NULL)) DESC;

</details>

6. <details><summary><b>My Read List</b>: Captures my current TBR list.</summary>

    ```sql
    SELECT Author, [Book Title]
    FROM Books
    WHERE Completed = FALSE AND [Purchased?] <> 'No'  # captures both physical and electronic titles
    ORDER BY Author;

</details>

7. <details><summary><b>Ongoing Series</b>: Lists all ongoing series. </summary>

    ```sql
   SELECT b.Author, b.[Book Series]
   FROM Books AS b INNER JOIN (SELECT Author, [Book Series], MIN([Book Title]) AS [My Book Title] FROM Books WHERE [Series Ongoing?] = TRUE GROUP BY Author, [Book Series])  AS subq ON (b.Author = subq.Author) AND (b.[Book Series] = subq.[Book Series]) AND (b.[Book Title] = subq.[My Book Title])
   WHERE b.[Series Ongoing?] = TRUE
   GROUP BY b.Author, b.[Book Series];

</details>

8. <details><summary><b>Overall Series</b>: Lists all series in the database.</summary>

      ```sql
      SELECT DISTINCT [Author], [Book Series]
      FROM Books
      WHERE [Book Series] IS NOT NULL 
            AND [Book Series] <> '' 
            AND [Book Series] NOT LIKE '*Collection*'
            AND [Book Series] NOT LIKE '*Standalone*'
      ORDER BY [Author], [Book Series];

</details>

9. <details><summary><b>Shopping List</b>: My book wish list. </summary>

     ```sql
    SELECT b.Author, b.[Book Title], b.[Book Series], b.Sequence, b.[Amazon Link], IIF(
            EXISTS(
                SELECT 1 
                FROM Books AS started
                WHERE started.[Book Series] = b.[Book Series] 
                AND started.Completed = TRUE 
                AND started.[Purchased?] <> 'No'
                AND started.[Book Series] NOT LIKE '*Standalone*'
                AND started.[Book Series] NOT LIKE '*Collection*'
            ), 
            'Yes', 
            'No'
        ) AS [Series Started]                    # books from already purchased series take precedence
    FROM Books AS b
    WHERE b.Completed = FALSE 
        AND b.[Purchased?] = 'No'
        AND b.[Released?] = TRUE
    GROUP BY b.Author, b.[Book Series], b.Sequence, b.[Book Title], b.[Amazon Link];

</details>

10. <details><summary><b>Tracking Purchases</b>: Captures all physical books I own and allows to keep track of pre-ordered titles pending delivery.</summary>
    
    ```sql
    SELECT Author, [Book Title], [Book Series], [Estimated Release Date]
    FROM Books
    WHERE [Purchased?] = 'Yes'
    ORDER BY Author;

</details>

11. <details><summary><b>Unreleased Titles</b>: Anticipated book releases in chronological order.</summary>

    ```sql
    SELECT Author, [Book Title], [Book Series], Sequence, [Estimated Release Date]
    FROM Books
    WHERE [Released?] = False
    ORDER BY IIf(IsDate([Estimated Release Date]), CDate([Estimated Release Date]), DateSerial(9999,12,31)), Author;

</details>


