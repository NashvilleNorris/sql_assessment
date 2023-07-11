# sql_assessment

    --1. The poetry in this database is the work of children in grades 1 through 5.  
    --a. How many poets from each grade are represented in the data?  

SELECT grade_id AS grade, COUNT (name) AS total_number

FROM author

GROUP BY grade_id 

ORDER BY grade_id

    --Grade    Total Number
    --1	       623
    --2	       1437
    --3	       2344
    --4	       3288
    --5	       3464

    --b. How many of the poets in each grade are Male and how many are Female? Only return the poets identified as Male or Female. 

SELECT gr.name AS grade_level, g.name AS gender, COUNT(a.name) AS total

FROM author AS a

INNER JOIN gender AS g

ON a.gender_id = g.id

INNER JOIN grade AS gr

ON a.grade_id = gr.id

WHERE g.id IN (1,2)

GROUP BY gr.name, g.name

ORDER BY gr.name

    -- Grade Level	Gender		Total Number
    -- "1st Grade"	"Female"	  243
    -- "1st Grade"	"Male"	    163
    -- "2nd Grade"	"Female"	  605
    -- "2nd Grade"	"Male"	    412
    -- "3rd Grade"	"Female"	  948
    -- "3rd Grade"	"Male"	    577
    -- "4th Grade"	"Female"	  1241
    -- "4th Grade"	"Male"	    723
    -- "5th Grade"	"Female"	  1294
    -- "5th Grade"	"Male"	    757

    --c. Do you notice any trends across all grades?
    --It seems as if there are more girls involved in writing poems than boys but regardless of gender, the total number continues to get bigger as the grade level goes up

    --2. Love and death have been popular themes in poetry throughout time. Which of these things do children write about more often? Which do they have the most to say about when they do? Return the **total number** of poems that mention **love** and **total number** that mention the word **death**, and return the **average character count** for poems that mention **love** and also for poems that mention the word **death**. Do this in a single query.	

WITH love_poems AS (
	
 SELECT id, title, char_count, poki_num, text
	
 FROM poem
	
 WHERE text LIKE '%love%'
	
 ),

death_poems AS 

 (
	
 SELECT id, title, char_count, poki_num, text
	
 FROM poem
	
 WHERE text LIKE '%death%'
	
 )

SELECT 'Love' AS theme, COUNT(*) AS total_number_of_poems, ROUND(AVG (char_count),2) AS ave_chara_in_poem

FROM love_poems

UNION

SELECT 'Death' AS theme, COUNT(*) AS total_number_of_poems, ROUND(AVG (char_count),2) AS ave_chara_in_poem

FROM death_poems

    --Theme		# Poems		Avg Character Count
    --"Death"	86	    	342.53
    --"Love"	4464		226.79


    --3. Do longer poems have more emotional intensity compared to shorter poems?  
    --a. Start by writing a query to return each emotion in the database with its average intensity and average character count. 

SELECT e.name AS theme, ROUND(AVG(p.char_count), 0) AS ave_character_count, CONCAT(ROUND(AVG(pe.intensity_percent),0),'%') AS ave_intensity_percentage

FROM poem AS p

INNER JOIN poem_emotion AS pe

ON p.id = pe.poem_id

INNER JOIN emotion AS e

ON pe.emotion_id = e.id

GROUP BY e.name

ORDER BY ave_character_count DESC

    -- Theme		Ave Character Count		Ave Intensity %
    -- "Fear"		256						"45%"
    -- "Sadness"	247						"39%"
    -- "Joy"		221						"48%"
    -- "Anger"		261						"44%"

    --Which emotion is associated the longest poems on average?

--Anger

    --Which emotion has the shortest?  

--Joy

--b. Convert the query you wrote in part a into a CTE. Then find the 5 most intense poems that express joy and whether they are to be longer or shorter than the average joy poem.   

WITH intensity AS (
		
  SELECT intensity_percent, emotion_id, poem_id
		
  FROM poem_emotion
		
  WHERE emotion_id = '4'
		
  ORDER BY intensity_percent DESC
		
  LIMIT 6
		
  ),

poem_title AS (
		
  SELECT title, char_count, text
		
  FROM poem
		
  ),
	
 ave_char_count AS (
		
  SELECT char_count, CASE
		
  WHEN char_count >= 221 THEN 'longer than average'
		
  WHEN char_count < 220 THEN 'less than average'
		
  ELSE 'average'
		
  END AS poem_length
		
  FROM poem
		
  INNER JOIN poem_emotion
		
  ON poem.id = poem_emotion.poem_id
		
  )
	
 SELECT title AS poem_title, CONCAT(intensity_percent, '%') AS intensity_percent, char_count, text AS poem
	
 FROM poem
	
 INNER JOIN intensity
	
 ON poem.id = intensity.poem_id
	
 INNER JOIN emotion
	
 ON intensity.emotion_id = emotion.id
	
 WHERE emotion_id = '4'
	
 GROUP BY title, char_count, intensity_percent, text
	
 ORDER BY intensity_percent DESC
	
 LIMIT 6
	
    --What is the most joyful poem about?

--It is about the author's dog

    --Do you think these are all classified correctly?

--I do not think they are classified correctly as there are some that are not joyful

--4. Compare the 5 most angry poems by 1st graders to the 5 most angry poems by 5th graders.  

SELECT CONCAT(a.grade_id, 'st') AS grade_level, g.name AS gender, CONCAT(pe.intensity_percent, '%') AS intensity_percent, e.name AS emotion_name, p.title AS poem_title, p.text

FROM author AS a

INNER JOIN poem AS p

ON a.id = p.author_id

INNER JOIN poem_emotion AS pe

ON p.id = pe.poem_id

INNER JOIN emotion AS e

ON pe.emotion_id = e.id

INNER JOIN gender AS g

ON a.gender_id = g.id

WHERE a.grade_id = '1'

AND e.name = 'Anger'

AND g.name IN ('Female','Male')

AND pe.intensity_percent > 75

ORDER BY intensity_percent DESC

LIMIT 5

    -- "1st"	"Female"	"86%"	"Anger"		"Pie"
    -- "1st"	"Female"	"83%"	"Anger"		"Nose is like a rose"
    -- "1st"	"Female"	"83%"	"Anger"		"I THINK OF YOU"
    -- "1st"	"Male"		"82%"	"Anger"		"CATTY"
    -- "1st"	"Male"		"79%"	"Anger"		"The Leaves Of Grace"


SELECT CONCAT(a.grade_id, 'th') AS grade_level, g.name AS gender, e.name AS emotion_name, CONCAT(pe.intensity_percent, '%') AS intensity_percent, p.title AS poem_title, p.text

FROM author AS a

INNER JOIN poem AS p

ON a.id = p.author_id

INNER JOIN poem_emotion AS pe

ON p.id = pe.poem_id

INNER JOIN emotion AS e

ON pe.emotion_id = e.id

INNER JOIN gender AS g

ON a.gender_id = g.id

WHERE a.grade_id = '5'

AND e.name = 'Anger'

AND g.name IN ('Female','Male')

AND pe.intensity_percent > 75

ORDER BY intensity_percent DESC

LIMIT 5

    -- "5th"	"Male"		"Anger"		"96%"	"Nature&apos;s Ways"
    -- "5th"	"Female"	"Anger"		"96%"	"Horse"
    -- "5th"	"Female"	"Anger"		"95%"	"Take Me Away"
    -- "5th"	"Female"	"Anger"		"95%"	"Think"
    -- "5th"	"Female"	"Anger"		"93%"	"Me! Me! and the world."

    --a. Which group writes the angriest poems according to the intensity score?  
--Females have the higher scores overall in terms of Anger Intensity

    --b. Who shows up more in the top five for grades 1 and 5, males or females?  
--Females appear more often than males

    --c. Which of these do you like the best?
--Horse by the 5th grader

		-- 5. Emily Dickinson was a famous American poet, who wrote many poems in the 1800s, examine the poets in the database with the name `emily`. Create a report showing the count of emilys by grade along with the distribution of emotions that characterize their work.

--I included the author_id column to show these were multiple entries by the same author

SELECT a.name, a.grade_id AS grade_level, p.author_id, e.name AS emotion

FROM author AS a

INNER JOIN poem AS p

ON a.id = p.author_id

INNER JOIN poem_emotion AS pe

ON p.id = pe.poem_id

INNER JOIN emotion AS e

ON pe.emotion_id = e.id

WHERE a.name ILIKE 'emily'

GROUP BY a.name, p.author_id, a.grade_id, e.name

ORDER BY a.grade_id
