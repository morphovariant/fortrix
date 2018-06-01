# Fortrix | Integrated Clinical Project Management

## Project Overview
Fortrix provides four major areas of functionality related to clinical project management: timeline management, enrollment forecasts, supply forecasts, and budgets.

### 1. Timeline management
CPMs manage very detailed timelines with Microsoft Project and Fortrix is not intended to replace these. This is partly due to the monthly nature of Fortrix (vs. the daily increments in Project), but primarily because the level of detail isn’t appropriate to the task of forecasting. 

Instead, Fortrix relies on eight high-level milestones that describe the general flow of all clinical trials: 
 
* __Planning__
* __Set-up__
* __FSA-FPI__ (First Site Active - First Patient In)
* __FPI-LPI__ (First Patient In - Last Patient In)
* __LPI-LPEoT__ (Last Patient In - Last Patient End of Treatment)
* __LPEoT-LPO__ (Last Patient End of Treatment - Last Patient Out)
* __LPO-DBL__ (Last Patient Out - Database Lock)
* __DBL-CSR__ (Database Lock - Clinical Study Report)
 
It is not uncommon for these eight milestones to span five or more years. It is also not uncommon for timelines to shift, both forward and backward, or to compress or extend particular milestones in response to test results. Therefore, ideal functionality would enable the CPMs to manage the months associated with each milestone of their studies within Fortrix, without requiring the system administrator (changes requiring the system administrator may take up to 48 hours). We achieved this with the combination of standard and dynamic Named Sets.

A Named Set is a custom subset of dimension members. Standard Named Sets are defined by an administrator by adding specific members to the set; any changes to the membership can only be made by the administrator.  Dynamic Named Sets are defined by an administrator with an MDX calculation which filters the dimension members based on criteria that CPMs can change. In the case of Timelines, the system administrator creates one standard Named Set for each study containing all months spanning the life of the study with a buffer at either end to give the CPM room to adjust. CPMs break up this five-year standard Named Set into milestones using the Timeline Template which enables them to manage the membership for each dynamic Named Set, one corresponding to each milestone.

In addition to granting CPMs some autonomy in the management of their studies, dynamic Named Sets simplify system configuration for the administrators. While a standard Named Set must be created specifically for each study, the eight dynamic Named Sets corresponding to timeline milestones can serve all studies, now and in the future. It is the context of the study that determines the membership of the set. In this way, dynamic named sets also help to reduce the effort needed to manage Fortrix templates, which typically combine standard elements with study-specific elements that must be carefully aligned when new templates are created for new studies. For example, the Enrollment Planning Template uses milestone Named Sets #3 and #4 as columns and study-specific standard Country and Cohort Named Sets as rows. 
 
### 2. Enrollment Forecasting
 
Enrollment forecasting is an art. CPMs must consider a number of variables including the prevalence of the target indication, the context of the geographic region under consideration, the availability of adequate clinical sites in those regions, the target number of subjects needed to confirm the results, and the high-level timeframes set by the study team. These variables interact with each other; for example, shorter time frame can be met by increasing the number of clinical sites. 

Within Fortrix, CPMs provide: 

* a target number of Sites for each Country 
* a target number of Subjects for each Country and Cohort
* an Enrollment Rate for each Country and Cohort
 
With this data, Fortrix performs and displays “helper calculations” which CPMs use as guides to forecasting Site activation and Subject enrollment. 

* __Site Activation Curves__ - given a target number of sites between 5-140, Fortrix supplies a monthly curve based on historical site activation on similarly sized studies.
* __Cumulative Sites__ - an accumulation of Sites activated
* __Percent of Target Sites__ - 75% of target is a guidepost for related activities.
* __Monthly Subjects__ - given the Cumulative Sites and an Enrollment Rate, this shows how many subjects we can reasonably expect to enroll each month
* __Cumulative Subjects__ - an accumulation of Subjects enrolled

Often, CPMs only have a total target Subjects and need to determine the best distribution of subjects across potential countries to meet enrollment timelines most efficiently. The Enrollment Template helps them to see how many, and by when, each country can reasonably enroll Subjects. If a country tends to enroll slowly or the indication is particularly rare, too few sites may mean the country is not an ideal choice for the study while another country may prove otherwise. This template makes these observations quickly apparent and makes it easy to adjust Sites and Subjects accordingly

Currently, system administrators must manage the membership of study-specific standard Named Sets for Countries and Cohorts. This specificity means maintaining separate templates for each study, which is not ideal. The reason for this is due to a bug in the underlying system which causes dynamic Named Sets in rows to interfere with the formulas placed in those rows (the helper calculations). Once this bug is fixed by the vendor, system administrators will need only maintain one Enrollment Template which will serve all studies; Time, Country, and Cohorts will change based on the selected study.

### 3. Treatment Planning
 
The purpose of a Cohort is to distinguish a group of Subjects by the treatment they are on. A treatment involves one or more Products, at a specified dose, on a specified schedule. Fortrix also supports the definition of randomized treatments, when the study model calls for it. CPMs can define one treatment regime for each cohort planned in the Enrollment forecast and Fortrix performs all of the calculations necessary to estimate supply need each Product, Country, and month. Fortrix aggregates these data across studies for the Supply team. 
 
Treatment planning is conceptually straightforward, but the calculations are among the most complex in the system. This is due to the fact that cancer therapies are typically administered on a schedule of repeated cycles. In the context of Treatment and Budget, these cycles represent a visit where a Subject will receive a treatment and undergo one or more lab tests (Lab Tests will be revisited in the Budget section). Additionally, the drug product is supplied in vials containing a certain amount of drug, but once a vial is used to treat one subject, even if drug remains, it cannot be administered to another subject (not even to the same subject at their next visit). Therefore, the calculation of supply must be resolved to whole number vials per Product, Country, and Cycle (visit). 
 
When a trial involves a combination therapy (more than one drug product given concurrently), the calculations require the counting of visits by Subject times the number of drug Products they are receiving in order to accurately calculate the number of vials needed of each. Fortrix’s dimensional model helps us here by enabling consistent calculation at the intersection of multiple dimensions including Country, Cohort, and Product. However, this is also a good example of the way Fortrix utilizes dimensions in an atypical fashion (see Atypical OLAP Modeling below) because an uninformed query of the model on Cycles involving a combination therapy would result in an inaccurate sum. For this reason, Cycles are also specially calculated for use in the Budget functionality as typically there is a cost associated with each visit.
 
### 4. Budget
 
From the Enrollment and Treatment planning, we derive the following accounts used to help estimate the monthly spend of the many contracts and collaborations that help to run large trials:
 
* __Enrollment of subjects__ - for costs incurred upon enrollment of an eligible subject
* __Activation of sites__ - for costs incurred upon successful activation of a clinical site
* __Cycles__ - for costs that tend to get invoiced with each visit
 
The Cycle curve is especially powerful and not something that was easily available previously. Fortrix makes it easy to combine the enrollment of Subjects over time, the estimated average number of months each Subject is expected to stay on study, and the average number of visits we expect them to make during that time. We can aggregate this across all Parts of a Study, or we can apply costs differently based on the Part of the Study a Subject is associated with. This latter is how we handle Lab Tests, the most time consuming part of managing a study budget. 
 
The tests are numerous and important; they are evidence of the tolerability and efficacy of an experimental medicine. The results of these tests are analyzed by biostatisticians, submitted to the FDA, and presented in scientific journals and at major conferences.  Many of these tests produce a sample which must be tracked and stored. Often, it is more efficient to batch the samples instead of sending them individually for evaluation in real-time. So, as with the art of Enrollment, Fortrix is able to produce and display a number of calculations that help the CPM determine the best batching schedule for individual tests and enter the associated costs into the budget accordingly.

## Summary of the major technological decisions made, including why
 
### Atypical OLAP Modeling

Before we chose an OLAP solution we conducted two separate RFPs (requests for proposals) to evaluate existing off-the-shelf tools, began work on a custom VBA application, and considered Python, VB.NET, and the UWP (Universal Windows Platform). The RFPs resulted in no viable options; most tools focus exclusively on supply and finance and enrollment was essential to our requirements. The VBA development was going well, but we quickly reached the extent of VBA UX capabilities and once we started considering even VB.NET, the door was open again for other alternatives. OLAP was not an obvious choice, but based on the demonstrations given by the vendor, we felt it was flexible enough to meet our needs and powerful enough to get us closer to stretch goals (like leveraging historical data) sooner than an in-house custom-coded application.
 
In order to use OLAP as a platform for active forecasting, it was necessary to adopt an atypical approach to OLAP modeling. OLAP is designed to take static historical transactional data which has been transformed specifically to fit the dimensional model; in doing so, users gain the ability to quickly analyze this data, to slice and dice it in innumerable ways, and to answer a variety of questions. Forecasting is not a static process, but it is also not transactional because it deals in monthly aggregates. With an atypical OLAP model, we were able to take advantage of the interconnectedness of a dimensional model while enabling a number of data entry points at various points in the model. 
 
The resulting cube will be comparatively sparse and reportable accounts will exist only at specific dimensional intersections, with the rest of the data serving as inputs to calculations supporting the reportable accounts.  For this reason, documentation and training will be essential. The dimensional model serves as both a major strength as well as a major weakness of the solution. It will be interesting to see how it stands the test of time.  

## Contact information
Jennifer Yoakum  
yoakumjm@uw.edu  
www.linkedin.com/in/jenniferyoakum

