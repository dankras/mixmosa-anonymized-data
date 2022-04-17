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
 
 9. `user_features`: derived table that aggregates all user attributes (e.g. the user's average physical attractiveness score from `picture_ratings`, their IQ score given their `responses` to intelligence `questions`, their neuroticism score given their `responses` to Big 5 `questions`).
    * features include: age, gender, attracted_to, iq_share_correct, agreeableness, conscientiousness, extraversion, neuroticism, openness, avg_physical_attractiveness_rating, height_inches, income_bucket, nicotine_score, alcohol_score, marijuana_score, has_used_psychedelics, behavioral_health_score, sex_partners_score, get_along_well_with_family_score, political_tolerance_score, ethnicity_importance_score, religion_importance_score, politics_importance_score, num_children_wanted_score, already_has_kids, casual_sex_score, hard_work_and_success_belief_score
 
 10. `interactions`: each row represents a date. derived from `swiping`. contains date outcomes (e.g. did the man like the woman? did the woman like the man?).
 
 
 ## Contact
 I'd be happy to answer any questions: first dot last on gmail.
 
