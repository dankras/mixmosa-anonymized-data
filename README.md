# Data from 2,961 In-Person Dates

## Context
Mixmosa was a dating service that organized 2,961 in-person dates. For each date, it observed:
1. the outcomes (e.g. did both people like each other?)
2. participant attributes (e.g. physical attractiveness, Big 5 Personality, intelligence, income, drug use, height, values, etc.)

I am sharing the (anonymized) dataset here because it may be of value to people interested in evolutionary psychology, machine learning, mate choice, dating & relationships, etc.

## Ideas
* [Here](https://dkras.substack.com/p/sex-differences-attractiveness-and) is an example of some analysis done using this dataset.
* Is it possible to predict the outcomes of first dates in advance?

## Quickstart
The dataset is available in two formats:
 1. CSV files
 2. Postgres SQL
 
 There are two tables I'd recommend you start with:
 1. `user_features` describes the participants: age, gender, sexual orientation, IQ, personality, physical attractiveness, etc.
 2. `interactions` describes the outcomes of each date: did the male like the female? did the female like the male? 
 
 Data was collected by running in-person speed dating events. Each date lasted 8 minutes.
 
 ## Tables
 1. `users`: people that created an account in the Mixmosa app
 
 2. `events`: speed dating event metadata (note that age ranges were not strictly enforced)
 
 3. `attendance`: which users went to which events 
 
 4. `event_feedback`: attendees were asked to rate their experience after each event on a 5-star scale
 
 5. `questions`: the set of questions users were required to complete before attending an event
 
 6. `responses`: users' answers to `questions`
 
 7. `swiping`: speed dating attendees were asked who they liked and disliked after 8-minute-long dates
 
 8. `picture_ratings`: users photos were scored on physical attractiveness by the opposite sex on a 1 - 10 scale, with 1 being least attractive, 5 being average, and 10 being most attractive. Independent raters were used (Mixmosa users did not rate other Mixmosa users).
 
 9. `user_features`: derived table that aggregates all user attributes (e.g. the user's average physical attractiveness score from `picture_ratings`, their IQ score given their `responses` to intelligence `questions`, their neuroticism score given their `responses` to Big 5 `questions`). See below for definitions for these features. You can also generate your own features from the raw data.
 
 10. `interactions`: each row represents a date. derived from `swiping`. contains date outcomes (e.g. did the man like the woman? did the woman like the man?).
 
 ## Feature Definitions in `user_features`

 **Self Explanatory**
 - `age`
 - `gender`
 - `attracted_to`
 - `height_inches`

 **Intelligence**
 - `iq_share_correct`: share of intelligence questions answered correctly by user
 
 **Big 5 Personality Traits**: 
- sum of individual question scores, with each question ranging from 1 to 3. Note that I decided to score "strongly agree" and "agree", and "strongly disagree" and "disagree" the same, which is why this is on a 3 point scale. If you'd like to take these differences into account and move to a 5 point scale, you can regenerate these features yourself. See code below.
- Specific features:
   - `agreeableness`
   - `conscientiousness`
   - `extraversion`
   - `neuroticism`
   - `openness`
   

```
   def get_personality_by_user(self):
        query = """
        select
            user_id,
            q.id as question_id,
            category,
            lower(misc->>'big_5') as big_5_category,
            subcategory,
            misc->>'key' as key,
            prompt,
            responses.id as responses_id,
            responses.text_response
        from responses
        inner join questions q on responses.question_id = q.id and q.category = 'Personality'
        inner join users u on responses.user_id = u.id;
        """
        personality_responses = pd.read_sql(query, self.engine)

        def score_personality_response(row):
            # TODO currently not differentiating between "strongly {agree, disagree}", and just {agree, disagree}
            response = row["text_response"]
            if row["key"] == "1":
                if response in ["Strongly Disagree", "Disagree"]:
                    return 1
                elif response == "Neutral":
                    return 2
                elif response in ["Strongly Agree", "Agree"]:
                    return 3
                else:
                    return None
            elif row["key"] == "-1":
                if response in ["Strongly Disagree", "Disagree"]:
                    return 3
                elif response == "Neutral":
                    return 2
                elif response in ["Strongly Agree", "Agree"]:
                    return 1
                else:
                    return None
            else:
                return None

        personality_responses["score"] = personality_responses.apply(
            score_personality_response, axis=1
        )

        personality_users = personality_responses.groupby(
            "user_id"
        ).responses_id.count()

        # currently 65 personality questions. filter for where users have answered all personality questions.
        users_completing_all_personality_questions = list(
            personality_users[personality_users >= 65].index
        )
        filtered_personality_responses = personality_responses[
            personality_responses.user_id.isin(
                users_completing_all_personality_questions
            )
        ]

        return pd.pivot_table(
            filtered_personality_responses,
            values="score",
            index=["user_id"],
            columns=["big_5_category"],
            aggfunc=np.sum,
        )
```

**Physical Attractiveness**
- `avg_physical_attractiveness_rating`: user's average physical attractiveness score from `picture_ratings` (1 - 10 scale)
 
**Income**: 
- `income_bucket`
 ```
   def score_income(income_bucket: str):
      if income_bucket == "Under $15,000":
            return 1
      elif income_bucket == "$15,000 - $24,999":
            return 2
      elif income_bucket == "$25,000 - $34,999":
            return 3
      elif income_bucket == "$35,000 - $49,999":
            return 4
      elif income_bucket == "$50,000 - $74,999":
            return 5
      elif income_bucket == "$75,000 - $99,999":
            return 6
      elif income_bucket == "$100,000 - $149,999":
            return 7
      elif income_bucket == "$150,000 - $199,999":
            return 8
      elif income_bucket == "$200,000 and over":
            return 9
      else:
            return None
 ```
**Drug Use**
 - `nicotine_score`:
```
   def score_nicotine(text_response):
      if text_response == "Never":
            return 0
      elif text_response in ["Daily", "Weekly", "Monthly"]:
            return 1
      else:
            return None
```
 - `alcohol_score`
 ```
   def score_alcohol(text_response):
      if text_response == "Never":
            return 0
      elif text_response == "Monthly":
            return 1
      elif text_response == "Weekly":
            return 2
      elif text_response == "Daily":
            return 3
      else:
            return None
 ```
 - `marijuana_score`
 ```
   def score_marijuana(text_response):
      if text_response == "Never":
            return 0
      elif text_response == "Monthly":
            return 1
      elif text_response == "Weekly":
            return 2
      elif text_response == "Daily":
            return 3
      else:
            return None
 ```
 - `has_used_psychedelics`
 ```
   psychedelics["has_used_psychedelics"] = psychedelics.text_response.apply(
      lambda r: 1 if r == "Yes" else 0
   )
 ```

 **Health**
 - `behavioral_health_score`: how strongly does the person agree with the following: i eat healthy, exercise, and am not overweight?
 ```
   def score_health(text_response):
      if text_response == "Strongly Disagree":
            return 0
      elif text_response == "Disagree":
            return 1
      elif text_response == "Neutral":
            return 2
      elif text_response == "Agree":
            return 3
      elif text_response == "Strongly Agree":
            return 4
      else:
            return None
 ```
 
 **Sex**
 - `sex_partners_score`
 ```
   def score_sex(text_response):
      if text_response == "0":
            return 0
      elif text_response == "1":
            return 1
      elif text_response == "2-4":
            return 2
      elif text_response == "5-9":
            return 3
      elif text_response == "10+":
            return 4
      else:
            return None
 ```
 - `casual_sex_score`: this is question 116, i.e.: "How strongly do you agree with the following: I do NOT want to have sex with a person until I am sure that we will have a long-term, serious relationship"
 ```
   def score_casual_sex(text_response):
      if text_response == "Strongly Disagree":
            return 4
      elif text_response == "Disagree":
            return 3
      elif text_response == "Neutral":
            return 2
      elif text_response == "Agree":
            return 1
      elif text_response == "Strongly Agree":
            return 0
      else:
            return None
 ```

**Kids**
 - `num_children_wanted_score`
 ```
   def score_num_children_wanted(text_response):
      if text_response == "0":
            return 0
      elif text_response == "1":
            return 1
      elif text_response == "2-3":
            return 2
      elif text_response == "4+":
            return 3
      else:
            return None
 ```
 - `already_has_kids`
 ```
   has_kids["already_has_kids"] = has_kids.text_response.apply(
      lambda r: 1 if r == "Yes" else 0
   )
 ```

 **Importance of {ethnicity, religion, politics} in a Romantic Partner**: Users were asked: How important is it that your romantic partner shares your {ethnicity, religion, political beliefs}?
 - `ethnicity_importance_score`
 - `religion_importance_score`
 - `politics_importance_score` 
```
def score_importance(text_response):
    if text_response == "Not Important":
        return 0
    elif text_response == "Somewhat Important":
        return 1
    elif text_response == "Very Important":
        return 2
    else:
        return None
```

 **Misc**
 - `political_tolerance_score`: this is question 179, i.e.: To what degree does the following phrase describe you? Comfortable being friends with someone that disagrees with me on important political topics.
 ```
   def score_political_tolerance(text_response):
      if text_response == "Strongly Disagree":
            return 0
      elif text_response == "Disagree":
            return 1
      elif text_response == "Neutral":
            return 2
      elif text_response == "Agree":
            return 3
      elif text_response == "Strongly Agree":
            return 4
      else:
            return None
 ```
 - `get_along_well_with_family_score`
 ```
   def score_family(text_response):
      if text_response in ["Strongly Disagree", "Disagree", "Neutral"]:
            return 0
      elif text_response in ["Agree", "Strongly Agree"]:
            return 1
      else:
            return None
 ```
- `hard_work_and_success_belief_score`
 ```
   def score_does_hard_work_lead_to_success(text_response):
      if text_response == "Strongly Disagree":
            return 0
      elif text_response == "Disagree":
            return 1
      elif text_response == "Neutral":
            return 2
      elif text_response == "Agree":
            return 3
      elif text_response == "Strongly Agree":
            return 4
      else:
            return None
 ```

 ## Contact
 I'd be happy to answer any questions: first dot last on gmail.
 
