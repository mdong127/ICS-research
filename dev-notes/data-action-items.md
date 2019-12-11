### Incorporate Courses / Classes API into Data pipeline 

1. where does `hashed path` get defined? in env.sh vs env.json?  
    - actually from `makeJson` which takes dirName (timestamp) as an argument and then creates `env.json`
    - bash file is unused
    - NEED `hashed` directory in UCBDATA
1. why does /research/UCBD2/pipeline-test still exist in env.json? bc env.json hasn't been updated from running master 
1. Double check that models retrain still runs on the hashed data in UCBD2/edw_data/hashed?  
1. Running pipeline should retrain all the models and write all the files service needs to /models 
    - need to copy over env.json & relaunch service?  Yes
1. Checking update-pipeline runs without error, run from your own directory & reserve /askoski for master?  Yes
    - need to run in screen?  Yes
1. how to verify pipeline has successfully run?  should have outputs in the timestamped dir?  
    - yes, or copy env.json and make sure service runs as expected
1. Missing files?? 
    - `/research/UCBDATA/edw_askoski_apr_student_requirements.txt "['PERSON_PARTY_SK'] not found in axis"`
    - `cp: cannot stat '/research/UCBDATA/edw_askoski_course_lists.txt': No such file or directory`
    - Models-AskOski/RNN/evaluate.py: `No such file or directory: '/research/UCBD2/classAPI/next_sem_classes_new.p'`
    - nearest_course.py: `FileNotFoundError: [Errno 2] No such file or directory: '/research/UCBD2/edw_data/2019-12-10-17-55/model/course2idx_new.json'`
    - what are all the _bk files & what file are these? 
        - mv: cannot stat '/research/UCBD2/edw_data/2019-12-10-17-55/model/course2idx_all.json': No such file or directory
        - mv: cannot stat '/research/UCBD2/edw_data/2019-12-10-17-55/model/idx2course_all.json': No such file or directory
        - mv: cannot stat '/research/UCBD2/edw_data/2019-12-10-17-55/model/askoski_new': No such file or directory
        - mv: cannot stat '/research/UCBD2/edw_data/2019-12-10-17-55/model/askoski_new.json': No such file or directory
    - File "../C2V/preprocess.py", line 190, in <module> with open(os.path.join(RNN_DIR, 'course2idx.json'), 'r', encoding='utf-8') as f:
        No such file or directory: '/research/UCBD2/edw_data/2019-12-10-17-55/model/course2idx.json' <- this file is missing 3 times
    -  need to copy from prev timestamped directory?  no, it was caused by missing directory classAPI which caused all the course mappings to not be generated 
1. Replace /home/askoski/Models-AskOski with /home/matthew/Models-AskOski - change back before creating PR

### Types of tasks / debugging

1. Incorporate fixed /research/UCBD2/classAPI/ into pipeline

- Requirements - not displaying unmet requirement bubble interface for some students.  This would be dealing with the APR object?
- Explore re-training is broken - currently uses old research data
- Revisit open seats daily poll of classes api in thread
- Creating a new filter in AskOski that will filter a user's suggested courses based on whether their majors fall within the class reservations. This filter will be applied automatically to the already existing Open Seats filter. This task is a two-step process:
1)  Modify Data-AskOski to include reserve capacity information with each class, which involves querying the reserved seats API and augmenting the next_sem_dict to include reserve capacity data.
2)  Modify Service-AskOski to filter classes shown based on the reserve capacity data and the student's own majors. This involves creating a mapping between majors as represented in AskOski to the requirement groups in the API, and then creating a filter with it.

---

# Completed 

### Indexing error that broke the system for entirety of F19 during the data-askoski transition:

- Service reads from the hashed majors, grades, etc. files which are in the top level of /research/UCBD2/edw_data 

These are older files from the end of summer. It’s decoding the student ids with /research/askoski_common/bins/sidHash_new.bin, which is being overwritten by the data pipeline with mappings calculated from newer data. (are all these things being mapped?)

- Service using old grade, majors, requirements tables + new sid lookup table -> indexing errors
- A really quick fix would be to run the pipeline on old data, which would revert the lookup table and fix the indexing errors on Service.
- I am in the process of pointing the data pipeline to write the lookups to a separate folder that wouldn’t mess with the current Service.

The problem was a combination of a couple issues:

1. some of the file names for campus data dumps were incorrect. not sure who was responsible for this but we should verify file names next time when writing stuff like this.
2. we were loading the ENV json and then rewriting it to disk, but it wasn't being reloaded in memory. we should have sanity checks next time in between stages of the pipeline to check things like this, or better yet, separate out each stage of the pipeline entirely.
3. importing modules caused the ENV json in each one to load at the start meaning it was stale after new json was created. again we should be using sanity checks at each stage to verify that assumptions made in the code are actually true.

## SQL DB Transition instead of pandas

- service-askoski will definitely have to change because it's will no longer load pickle files but make SQL queries?  possibly models-askoski will change
- keep data askoski pipeline to process data but load exports into mySQL to be used in production


### Move all scripts and post-processed data out of UCBDATA & UCBD2
    
    - *Don't delete the encrypted files o/w you have to wait for the next data dump*
    - verify data is encrypted before moving out of UCBDATA - actually just keep here according to Zach
    - which are the files to not delete - don't delete the raw files

1. [x] archive files in UCBD2
1. [x] remove folders manually from UCBD2 - check if they can be deleted
    - pycache is just machine optimized code created every time python script is run, can be ignored - or deleted in this case
    - models-askoski is deprecated, nothing in git log
    - dev is empty
1. [x] change default srcDir path in models and add search model retrain
1. [x] Double save hash in UCBD2 & remove hashed data from UCBDATA in refresh.py 
    - Double check `rm -r` works from `os.system`
1. PR note
Included:
- Double save anonymized data to UCBD2/edw_data/hashed
- Remove hashed data from UCBDATA after saving to UCBD2

Test:
- Running pipeline should have hashed data saved to both timestamped directory and hashed directory in edw_data as well as hashed data removed from from UCBDATA