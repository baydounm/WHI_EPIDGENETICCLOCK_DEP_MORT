cd "E:\WHI_EPIGENETICCLOCK_DEP\DATA"

capture log close
capture log using "E:\WHI_EPIGENETICCLOCK_DEP\OUTPUT\DATA_MANAGEMENT.smcl", replace

use WHI_EPIGENETICCLOCK_DEP,clear
sort ID
save, replace

use WHI_EPIGENETICCLOCKDATA,clear
sort ID
save, replace

use Followup_time_WHI,clear
keep ID ENDFOLLOWDY
save Followup_time_WHI_small, replace
sort ID
save, replace


//STEP 1: MERGE INITIAL DATASET WITH EPIGENETIC CLOCK DATASET//

use WHI_EPIGENETICCLOCK_DEP,clear
merge ID using WHI_EPIGENETICCLOCKDATA
save WHI_EPIGENETICCLOCK_DEPfin, replace
capture drop _merge
sort ID
save WHI_EPIGENETICCLOCK_DEPfin, replace
merge ID using Followup_time_WHI_small
save WHI_EPIGENETICCLOCK_DEPfin, replace



capture log close
capture log using "E:\WHI_EPIGENETICCLOCK_DEP\OUTPUT\FIGURE1.smcl",replace


//STEP 2: FIGURE 1: PARTICIPANT FLOWCHART//////////////

**SAMPLE 1 WITH COMPLETE DEMOGRAPHICS**

capture drop sample1
gen sample1=.
replace sample1=1 if AGE~=. & RACE~=. & RACE~=9
replace sample1=0 if sample1~=1

tab sample1

**SAMPLE 2 WITH COMPLETE EPIGENETIC CLOCKS and DEMOGRAPHICS DATA**

capture drop EEAA
gen EEAA=eeaa

capture drop IEAA
gen IEAA=ieaa

capture drop AgeAccelPheno
gen AgeAccelPheno=ageaccelpheno

capture drop AgeAccelGrimAge
gen AgeAccelGrimAge=ageaccelgrim

capture drop sample2
gen sample2=.
replace sample2=1 if sample1==1 & EEAA~=. & IEAA~=. & AgeAccelPheno~=. & AgeAccelGrimAge~=.
replace sample2=0 if sample2~=1

tab sample2

**SAMPLE 3 WITH COMPLETE DEPRESSION and ANTIDEPRESSANT DATA**

capture drop sample3
gen sample3=. 
replace sample3=1 if sample2==1 & DEPRESSION_BASELINE_BR~=. & ANTIDEPRESSANT_BR~=.
replace sample3=0 if sample3~=1


capture drop sample_final
gen sample_final=sample3

save, replace


capture drop RACEBR
gen RACEBR=.
replace RACEBR=0 if RACE==5
replace RACEBR=1 if RACE==4
replace RACEBR=2 if RACE~=4 & RACE~=5 & RACE~=9

recode INCOME_BR (99=.)

save, replace


capture log close
capture log using "E:\WHI_EPIGENETICCLOCK_DEP\OUTPUT\IMPUTATIONS.smcl",replace


//STEP 3: MULTIPLE IMPUTATIONS: //////////////



use WHI_EPIGENETICCLOCK_DEPfin, clear

save finaldata_unimputed, replace

sort ID 

save, replace

set matsize 11000



use finaldata_unimputed, clear


keep ID EEAA IEAA AgeAccelPheno	AgeAccelGrimAge DEP* ANTI* DEATH_EVENT DEATH_TTE WHI_CT_OS AGE AGE_BR RACEBR ETHNICITY MARITAL_BR EDUC_BR INCOME_BR SMOKING_BR ALCOHOL_BR PHYSICAL_ACTIVITY_MET BMI BMIC_BR CVD_BR12 HT_BR12 DIAB_BR12 HYPERLIPIDEMIA_BR SRH_BR sample1 sample2 sample_final ENDFOLLOWDY

keep if sample_final==1


save finaldata_unimputedfin, replace

capture mi set, clear
capture mi stset, clear


mi set flong

capture drop DEATH_AGE
gen DEATH_AGE=AGE+DEATH_TTE/365.25

capture drop AGE_EVENT
gen AGE_EVENT=DEATH_AGE if DEATH_EVENT==1
replace AGE_EVENT=AGE+(ENDFOLLOWDY/365.25) if DEATH_EVENT==0

save, replace


mi unregister MARITAL_BR EDUC_BR INCOME_BR SMOKING_BR ALCOHOL_BR  BMI CVD_BR12 HT_BR12 DIAB_BR12 HYPERLIPIDEMIA_BR SRH_BR PHYSICAL_ACTIVITY_MET RACEBR ETHNICITY AGE sample_final
mi register imputed MARITAL_BR EDUC_BR INCOME_BR SMOKING_BR  ALCOHOL_BR BMI CVD_BR12 HT_BR12 DIAB_BR12 HYPERLIPIDEMIA_BR SRH_BR PHYSICAL_ACTIVITY_MET 
mi register passive  ETHNICITY 


mi stset AGE_EVENT, failure(DEATH_EVENT==1) enter(AGE) origin(AGE) id(ID) scale(1)


***************VARIABLES TO BE IMPUTED*************************************
***MARITAL_BR EDUC_BR INCOME_BR SMOKING_BR  BMI CVD_BR12 HT_BR12 DIAB_BR12 HYPERLIPIDEMIA_BR SRH_BR PHYSICAL_ACTIVITY_MET
***VARIABLES TO BE USED IN THE IMPUTATION: AGE RACEBR ETHNICITY



mi impute chained (mlogit) MARITAL_BR EDUC_BR INCOME_BR SMOKING_BR ALCOHOL_BR CVD_BR12 HT_BR12 DIAB_BR12 HYPERLIPIDEMIA_BR SRH_BR (regress) BMI PHYSICAL_ACTIVITY_MET = AGE i.ETHNICITY i.RACEBR if  sample_final==1 , force augment noisily  add(5) rseed(1234) savetrace(tracefile, replace) 



save finaldata_imputed, replace


save finaldata_imputedFINAL, replace

bysort  BMIC_BR: su BMI, det

capture drop BMIC_BR2
mi passive: gen BMIC_BR2=.
mi passive: replace BMIC_BR2=1 if BMI<25
mi passive: replace BMIC_BR2=2 if BMI>=25 & BMI<30
mi passive: replace BMIC_BR2=3 if BMI>=30 & BMI~=.

mi estimate: prop BMIC_BR2 if sample_final==1

save finaldata_imputedFINAL, replace

capture log close
capture log using "E:\WHI_EPIGENETICCLOCK_DEP\OUTPUT\FIGURE1.smcl",replace


//////////////////////////////////STDESCRIBE and STSUM//////////////////////////////////////////////////////////////////////////////

use finaldata_imputedFINAL,clear

mi extract 0

save finaldata_unimputed_extract0, replace

stdescribe if sample_final==1
stsum if sample_final==1


foreach x of varlist DEPRESSION_BASELINE_BR ANTIDEPRESSANT_BR DEP_ANTIDEP_DICH {
sts test `x' if sample_final==1	
}



sts graph if sample_final==1, by(DEPRESSION_BASELINE_BR) risktable
graph save "E:\WHI_EPIGENETICCLOCK_DEP\OUTPUT\FIGURE1Bfin.gph",replace

sts graph  if sample_final==1, by(ANTIDEPRESSANT_BR) risktable
graph save "E:\WHI_EPIGENETICCLOCK_DEP\OUTPUT\FIGURE1Cfin.gph",replace


sts graph  if sample_final==1, by(DEP_ANTIDEP_DICH) risktable
graph save "E:\WHI_EPIGENETICCLOCK_DEP\OUTPUT\FIGURE1Dfin.gph",replace
	

graph bar (mean) DEPRESSION_BASELINE_BR (mean) ANTIDEPRESSANT_BR (mean) DEP_ANTIDEP_DICH if sample_final==1
graph save "E:\WHI_EPIGENETICCLOCK_DEP\OUTPUT\FIGURE1Afin.gph",replace
	
save, replace 

capture log close
capture log using "E:\WHI_EPIGENETICCLOCK_DEP\OUTPUT\TABLE1.smcl",replace

///////////////////////////TABLE 1: Associations of sociodemographic, lifestyle and health characteristics with epigenetic age acceleration (n=) – Women's Health Initiative/////////////////


**EEAA	
**IEAA	
**AgeAccelPheno	
**AgeAccelGrimAge


**DEPRESSION_BASELINE_BR
**DEPRESSION_3YR_BR
**DEPRESSION_PATTERN
**ANTIDEPRESSANT_BR
**ANTIDEPRESSANT_BR_3YR
**ANTIDEPRESSANT_BASELINE_3YR
**DEP_ANTIDEP_DICH
**DEP_ANTIDEP_DICH_3YRS
**DEATH_EVENT
**DEATH_TTE
**WHI_CT_OS



**AGE 
**AGE_BR
**RACEBR
**ETHNICITY
**MARITAL_BR
**EDUC_BR
**INCOME_BR 
**SMOKING_BR 
**ALCOHOL_BR 
**PHYSICAL_ACTIVITY_MET 
**BMI 
**BMIC_BR 
**CVD_BR12 
**HT_BR12 
**DIAB_BR12 
**HYPERLIPIDEMIA_BR 
**SRH_BR

use finaldata_imputedFINAL,clear



************OVERALL DISTRIBUTION****************

foreach x of varlist  AGE_BR RACEBR ETHNICITY MARITAL_BR EDUC_BR INCOME_BR SMOKING_BR  ALCOHOL_BR BMIC_BR2 CVD_BR12 HT_BR12 DIAB_BR12 HYPERLIPIDEMIA_BR SRH_BR DEPRESSION_BASELINE_BR {
	mi estimate: prop `x' if sample_final==1
}


foreach x of varlist AGE BMI  PHYSICAL_ACTIVITY_MET EEAA IEAA AgeAccelPheno AgeAccelGrimAge {
	mi estimate: mean `x' if sample_final==1
	
	}

	
	
******************ASSOCIATION WITH EACH EPIGENETIC CLOCK***************

foreach x of varlist  AGE_BR RACEBR ETHNICITY MARITAL_BR EDUC_BR INCOME_BR SMOKING_BR ALCOHOL_BR  BMIC_BR2 CVD_BR12 HT_BR12 DIAB_BR12 HYPERLIPIDEMIA_BR SRH_BR DEPRESSION_BASELINE_BR {
	foreach y of varlist EEAA IEAA AgeAccelPheno AgeAccelGrimAge {
	mi estimate: reg  `y' i.`x' if sample_final==1
}
}

foreach x of varlist AGE BMI  PHYSICAL_ACTIVITY_MET {
	foreach y of varlist EEAA IEAA AgeAccelPheno AgeAccelGrimAge {
	mi estimate: reg  `y' `x' if sample_final==1
}
}


save, replace

capture log close


capture log close
capture log using "E:\WHI_EPIGENETICCLOCK_DEP\OUTPUT\TABLE2.smcl",replace


///////////////////////////TABLE 2: Associations of sociodemographic, lifestyle and health characteristics with depression, antidepression use and all-cause mortality risk (n=) – Women's Health Initiative////////////////////////

use finaldata_imputedFINAL,clear

mi stset, clear

mi stset AGE_EVENT, failure(DEATH_EVENT==1) enter(AGE) origin(AGE) id(ID) scale(1)


***************Y=DEPRESSION_BASELINE_BR***********************************

foreach x of varlist AGE BMI PHYSICAL_ACTIVITY_MET {
		mi estimate: mlogit  DEPRESSION_BASELINE_BR `x' if sample_final==1
}



foreach x of varlist AGE_BR RACEBR ETHNICITY MARITAL_BR EDUC_BR INCOME_BR SMOKING_BR ALCOHOL_BR  BMIC_BR2 CVD_BR12 HT_BR12 DIAB_BR12 HYPERLIPIDEMIA_BR SRH_BR {
		mi estimate: mlogit  DEPRESSION_BASELINE_BR i.`x' if sample_final==1
}


*****************Y=ANTIDEPRESSANT_BR**************************************


foreach x of varlist AGE BMI PHYSICAL_ACTIVITY_MET {
		mi estimate: mlogit  ANTIDEPRESSANT_BR `x' if sample_final==1
}



foreach x of varlist AGE_BR RACEBR ETHNICITY MARITAL_BR EDUC_BR INCOME_BR SMOKING_BR ALCOHOL_BR  BMIC_BR2 CVD_BR12 HT_BR12 DIAB_BR12 HYPERLIPIDEMIA_BR SRH_BR {
		mi estimate: mlogit  ANTIDEPRESSANT_BR i.`x' if sample_final==1
}

**************Y=DEP_ANTIDEP_DICH*****************************************


foreach x of varlist AGE BMI PHYSICAL_ACTIVITY_MET {
		mi estimate: mlogit  DEP_ANTIDEP_DICH `x' if sample_final==1
}



foreach x of varlist AGE_BR RACEBR ETHNICITY MARITAL_BR EDUC_BR INCOME_BR SMOKING_BR ALCOHOL_BR  BMIC_BR2 CVD_BR12 HT_BR12 DIAB_BR12 HYPERLIPIDEMIA_BR SRH_BR {
		mi estimate: mlogit  DEP_ANTIDEP_DICH i.`x' if sample_final==1
}


*****************Y=ALL-CAUSE MORTALITY************************************

foreach x of varlist AGE BMI PHYSICAL_ACTIVITY_MET {
		mi estimate: stcox `x' if sample_final==1
}


foreach x of varlist AGE_BR RACEBR ETHNICITY MARITAL_BR EDUC_BR INCOME_BR SMOKING_BR ALCOHOL_BR  BMIC_BR2 CVD_BR12 HT_BR12 DIAB_BR12 HYPERLIPIDEMIA_BR SRH_BR {
		mi estimate: stcox i.`x' if sample_final==1
}


capture log close
capture log using "E:\WHI_EPIGENETICCLOCK_DEP\OUTPUT\TABLE3.smcl",replace


/////////////////TABLE 3: Logistic regression models for change in depression, antidepressant use as well as depression and/or antidepressant use as predictors of estimates of epigenetic age acceleration//////

************Y=DEPRESSION_BASELINE_BR***********************

***********MODEL 1***************

foreach x of varlist IEAA EEAA AgeAccelPheno AgeAccelGrimAge {
mi estimate: mlogit DEPRESSION_BASELINE_BR `x' if sample_final==1

}


***********MODEL 2: ADJUSTED FOR DEMOGRAPHICS***************

foreach x of varlist IEAA EEAA AgeAccelPheno AgeAccelGrimAge {
mi estimate: mlogit DEPRESSION_BASELINE_BR `x' i.AGE_BR i.RACEBR i.ETHNICITY i.MARITAL_BR i.EDUC_BR i.INCOME_BR if sample_final==1

}

**********Model 3*********************************


foreach x of varlist IEAA EEAA AgeAccelPheno AgeAccelGrimAge {
mi estimate: mlogit DEPRESSION_BASELINE_BR `x' i.AGE_BR i.RACEBR i.ETHNICITY i.MARITAL_BR i.EDUC_BR i.INCOME_BR i.SMOKING_BR i.ALCOHOL_BR  i.BMIC_BR2 i.CVD_BR12 i.HT_BR12 i.DIAB_BR12 i.HYPERLIPIDEMIA_BR i.SRH_BR PHYSICAL_ACTIVITY_MET if sample_final==1

}

*************Y=ANTIDEPRESSANT_BR******************

***********MODEL 1***************

foreach x of varlist IEAA EEAA AgeAccelPheno AgeAccelGrimAge {
mi estimate: mlogit ANTIDEPRESSANT_BR `x' if sample_final==1

}


***********MODEL 2: ADJUSTED FOR DEMOGRAPHICS***************

foreach x of varlist IEAA EEAA AgeAccelPheno AgeAccelGrimAge {
mi estimate: mlogit ANTIDEPRESSANT_BR `x' i.AGE_BR i.RACEBR i.ETHNICITY i.MARITAL_BR i.EDUC_BR i.INCOME_BR if sample_final==1

}

**********Model 3*********************************


foreach x of varlist IEAA EEAA AgeAccelPheno AgeAccelGrimAge {
mi estimate: mlogit ANTIDEPRESSANT_BR `x' i.AGE_BR i.RACEBR i.ETHNICITY i.MARITAL_BR i.EDUC_BR i.INCOME_BR i.SMOKING_BR i.ALCOHOL_BR  i.BMIC_BR2 i.CVD_BR12 i.HT_BR12 i.DIAB_BR12 i.HYPERLIPIDEMIA_BR i.SRH_BR PHYSICAL_ACTIVITY_MET if sample_final==1

}


*************Y=DEP_ANTIDEP_DICH******************

***********MODEL 1***************

foreach x of varlist IEAA EEAA AgeAccelPheno AgeAccelGrimAge {
mi estimate: mlogit DEP_ANTIDEP_DICH `x' if sample_final==1

}


***********MODEL 2: ADJUSTED FOR DEMOGRAPHICS***************

foreach x of varlist IEAA EEAA AgeAccelPheno AgeAccelGrimAge {
mi estimate: mlogit DEP_ANTIDEP_DICH `x' i.AGE_BR i.RACEBR i.ETHNICITY i.MARITAL_BR i.EDUC_BR i.INCOME_BR if sample_final==1

}

**********Model 3*********************************


foreach x of varlist IEAA EEAA AgeAccelPheno AgeAccelGrimAge {
mi estimate: mlogit DEP_ANTIDEP_DICH `x' i.AGE_BR i.RACEBR i.ETHNICITY i.MARITAL_BR i.EDUC_BR i.INCOME_BR i.SMOKING_BR i.ALCOHOL_BR  i.BMIC_BR2 i.CVD_BR12 i.HT_BR12 i.DIAB_BR12 i.HYPERLIPIDEMIA_BR i.SRH_BR PHYSICAL_ACTIVITY_MET if sample_final==1

}


capture log close
capture log using "E:\WHI_EPIGENETICCLOCK_DEP\OUTPUT\TABLE4.smcl",replace

***********PRELIMINARY DATA MANAGEMENT************

capture drop DEPRESSION_PATTERN2
gen DEPRESSION_PATTERN2=DEPRESSION_PATTERN
recode DEPRESSION_PATTERN2 (3=0) (1=2) (2=1)


capture drop ANTIDEP_PATTERN
gen ANTIDEP_PATTERN=.
replace ANTIDEP_PATTERN=0 if (ANTIDEPRESSANT_BR==0 & ANTIDEPRESSANT_BR_3YR==0) | (ANTIDEPRESSANT_BR==1 & ANTIDEPRESSANT_BR_3YR==1)
replace ANTIDEP_PATTERN=1 if (ANTIDEPRESSANT_BR==1 & ANTIDEPRESSANT_BR_3YR==0) 
replace ANTIDEP_PATTERN=1 if (ANTIDEPRESSANT_BR==2 & ANTIDEPRESSANT_BR_3YR==1)

capture drop ANTIDEP_DEPRESSION_PATTERN
gen ANTIDEP_DEPRESSION_PATTERN=.
replace ANTIDEP_DEPRESSION_PATTERN=0 if (DEP_ANTIDEP_DICH==0 & DEP_ANTIDEP_DICH_3YRS==0) |(DEP_ANTIDEP_DICH==1 & DEP_ANTIDEP_DICH_3YRS==1)
replace ANTIDEP_DEPRESSION_PATTERN=1 if (DEP_ANTIDEP_DICH==1 & DEP_ANTIDEP_DICH_3YRS==0) |(DEP_ANTIDEP_DICH==1 & DEP_ANTIDEP_DICH_3YRS==0)
replace ANTIDEP_DEPRESSION_PATTERN=2 if (DEP_ANTIDEP_DICH==0 & DEP_ANTIDEP_DICH_3YRS==1) |(DEP_ANTIDEP_DICH==0 & DEP_ANTIDEP_DICH_3YRS==1)

**0: no change*
**1: Decrease
**2: Increase


foreach x of varlist DEPRESSION_BASELINE_BR ANTIDEPRESSANT_BR DEP_ANTIDEP_DICH   DEPRESSION_PATTERN2 ANTIDEP_PATTERN ANTIDEP_DEPRESSION_PATTERN {
mi xeq 0: stdescribe if `x'~=. & sample_final==1

} 

foreach x of varlist DEPRESSION_BASELINE_BR ANTIDEPRESSANT_BR DEP_ANTIDEP_DICH   DEPRESSION_PATTERN2 ANTIDEP_PATTERN ANTIDEP_DEPRESSION_PATTERN {
mi xeq 0: stsum if `x'~=. & sample_final==1
} 


foreach x of varlist DEPRESSION_BASELINE_BR ANTIDEPRESSANT_BR DEP_ANTIDEP_DICH   DEPRESSION_PATTERN2 ANTIDEP_PATTERN ANTIDEP_DEPRESSION_PATTERN {
	mi estimate: prop `x' if sample_final==1
} 


///////////////////////////TABLE 4: Cox proportional hazard models for depression, antidepressant use, depression and/or antidepressant use as well as epigenetic age acceleration predictors of all-cause mortality risks ///////


***********MODEL 1***************

foreach x of varlist IEAA EEAA AgeAccelPheno AgeAccelGrimAge DEPRESSION_BASELINE_BR ANTIDEPRESSANT_BR DEP_ANTIDEP_DICH DEPRESSION_PATTERN2 ANTIDEP_PATTERN ANTIDEP_DEPRESSION_PATTERN {
mi estimate: stcox  `x' if sample_final==1

}



foreach x of varlist   DEPRESSION_PATTERN2 ANTIDEP_PATTERN ANTIDEP_DEPRESSION_PATTERN {
mi estimate: stcox  i.`x' if sample_final==1

}


***********MODEL 2: ADJUSTED FOR DEMOGRAPHICS***************

foreach x of varlist IEAA EEAA AgeAccelPheno AgeAccelGrimAge DEPRESSION_BASELINE_BR ANTIDEPRESSANT_BR   {
mi estimate: stcox `x' i.AGE_BR i.RACEBR i.ETHNICITY i.MARITAL_BR i.EDUC_BR i.INCOME_BR if sample_final==1

}


foreach x of varlist   DEPRESSION_PATTERN2 ANTIDEP_PATTERN ANTIDEP_DEPRESSION_PATTERN {
mi estimate: stcox i.`x' i.AGE_BR i.RACEBR i.ETHNICITY i.MARITAL_BR i.EDUC_BR i.INCOME_BR if sample_final==1

}


**********Model 3*********************************


foreach x of varlist IEAA EEAA AgeAccelPheno AgeAccelGrimAge DEPRESSION_BASELINE_BR ANTIDEPRESSANT_BR   {
mi estimate: stcox  `x' i.AGE_BR i.RACEBR i.ETHNICITY i.MARITAL_BR i.EDUC_BR i.INCOME_BR i.SMOKING_BR i.ALCOHOL_BR  i.BMIC_BR2 i.CVD_BR12 i.HT_BR12 i.DIAB_BR12 i.HYPERLIPIDEMIA_BR i.SRH_BR PHYSICAL_ACTIVITY_MET if sample_final==1

}


foreach x of varlist  DEPRESSION_PATTERN2 ANTIDEP_PATTERN ANTIDEP_DEPRESSION_PATTERN {
mi estimate: stcox  i.`x' i.AGE_BR i.RACEBR i.ETHNICITY i.MARITAL_BR i.EDUC_BR i.INCOME_BR i.SMOKING_BR i.ALCOHOL_BR  i.BMIC_BR2 i.CVD_BR12 i.HT_BR12 i.DIAB_BR12 i.HYPERLIPIDEMIA_BR i.SRH_BR PHYSICAL_ACTIVITY_MET if sample_final==1

}

capture log close

capture log using "E:\WHI_EPIGENETICCLOCK_DEP\OUTPUT\TABLE5.smcl",replace


////////////////////////TABLE 5: Causal mediation analysis for change in depression, antidepressant use as well as depression and/or antidepressant use as mediators and/or moderators of the relationship between estimates of epigenetic age acceleration and all-cause mortality risks ///////




 
 
capture drop AGE_BRg* 
tab AGE_BR,generate(AGE_BRg)


capture drop RACEBRg*
tab RACEBR, generate(RACEBRg)

capture drop ETHNICITYg*
tab ETHNICITY, generate(ETHNICITYg)

capture drop MARITAL_BRg*
tab MARITAL_BR, generate(MARITAL_BRg)

capture drop EDUC_BRg*
tab EDUC_BR, generate(EDUC_BRg)

capture drop INCOME_BRg*
tab INCOME_BR, generate(INCOME_BRg)

capture drop SMOKING_BRg*
tab SMOKING_BR, generate(SMOKING_BRg)
 
capture drop ALCOHOL_BRg*
tab ALCOHOL_BR, generate(ALCOHOL_BRg)

capture drop BMIC_BR2g*
tab BMIC_BR2, generate(BMIC_BR2g)

capture drop  CVD_BR12g*
tab CVD_BR12, generate(CVD_BR12g)

capture drop HT_BR12g*
tab HT_BR12, generate(HT_BR12g)

capture drop DIAB_BR12g*
tab DIAB_BR12, generate(DIAB_BR12g)

capture drop HYPERLIPIDEMIA_BRg*
tab HYPERLIPIDEMIA_BR, generate(HYPERLIPIDEMIA_BRg)

capture drop SRH_BRg*
tab SRH_BR, generate(SRH_BRg)


capture drop zIEAA zEEAA zAgeAccelPheno zAgeAccelGrimAge
foreach x of varlist IEAA EEAA AgeAccelPheno AgeAccelGrimAge {
	egen z`x'=std(`x') if sample_final==1
}

*************************************MED4WAY******************************************


**MODEL 1**

foreach m of varlist zIEAA zEEAA zAgeAccelPheno zAgeAccelGrimAge {
mi estimate, cmdok esampvaryok: med4way DEPRESSION_BASELINE_BR `m'    if sample_final==1 , a0(0) a1(1) m(0) yreg(cox) mreg(linear) 
}


foreach m of varlist zIEAA zEEAA zAgeAccelPheno zAgeAccelGrimAge {
mi estimate, cmdok esampvaryok: med4way ANTIDEPRESSANT_BR `m'    if sample_final==1 , a0(0) a1(1) m(0) yreg(cox) mreg(linear) 
}


foreach m of varlist zIEAA zEEAA zAgeAccelPheno zAgeAccelGrimAge {
mi estimate, cmdok esampvaryok: med4way DEP_ANTIDEP_DICH `m'    if sample_final==1 , a0(0) a1(1) m(0) yreg(cox) mreg(linear) 
  
}


**MODEL 2**

foreach m of varlist zIEAA zEEAA zAgeAccelPheno zAgeAccelGrimAge {
mi estimate, cmdok esampvaryok: med4way DEPRESSION_BASELINE_BR `m' AGE_BRg* RACEBRg* ETHNICITYg* MARITAL_BRg* EDUC_BRg* INCOME_BRg*   if sample_final==1 , a0(0) a1(1) m(0) yreg(cox) mreg(linear) 
}


foreach m of varlist zIEAA zEEAA zAgeAccelPheno zAgeAccelGrimAge {
mi estimate, cmdok esampvaryok: med4way ANTIDEPRESSANT_BR `m' AGE_BRg* RACEBRg* ETHNICITYg* MARITAL_BRg* EDUC_BRg* INCOME_BRg*   if sample_final==1 , a0(0) a1(1) m(0) yreg(cox) mreg(linear) 
}


foreach m of varlist zIEAA zEEAA zAgeAccelPheno zAgeAccelGrimAge {
mi estimate, cmdok esampvaryok: med4way DEP_ANTIDEP_DICH `m' AGE_BRg* RACEBRg* ETHNICITYg* MARITAL_BRg* EDUC_BRg* INCOME_BRg*   if sample_final==1 , a0(0) a1(1) m(0) yreg(cox) mreg(linear) 
  
}



**MODEL 3**

foreach m of varlist zIEAA zEEAA zAgeAccelPheno zAgeAccelGrimAge {
mi estimate, cmdok esampvaryok: med4way DEPRESSION_BASELINE_BR `m' AGE_BRg* RACEBRg* ETHNICITYg* MARITAL_BRg* EDUC_BRg* INCOME_BRg* SMOKING_BRg* ALCOHOL_BRg*  BMIC_BR2g* CVD_BR12g* HT_BR12g* DIAB_BR12g* HYPERLIPIDEMIA_BRg* SRH_BRg* PHYSICAL_ACTIVITY_MET  if sample_final==1 , a0(0) a1(1) m(0) yreg(cox) mreg(linear) 
}


foreach m of varlist zIEAA zEEAA zAgeAccelPheno zAgeAccelGrimAge {
mi estimate, cmdok esampvaryok: med4way ANTIDEPRESSANT_BR `m' AGE_BRg* RACEBRg* ETHNICITYg* MARITAL_BRg* EDUC_BRg* INCOME_BRg* SMOKING_BRg* ALCOHOL_BRg*  BMIC_BR2g* CVD_BR12g* HT_BR12g* DIAB_BR12g* HYPERLIPIDEMIA_BRg* SRH_BRg* PHYSICAL_ACTIVITY_MET  if sample_final==1 , a0(0) a1(1) m(0) yreg(cox) mreg(linear) 
}


foreach m of varlist zIEAA zEEAA zAgeAccelPheno zAgeAccelGrimAge {
mi estimate, cmdok esampvaryok: med4way DEP_ANTIDEP_DICH `m' AGE_BRg* RACEBRg* ETHNICITYg* MARITAL_BRg* EDUC_BRg* INCOME_BRg* SMOKING_BRg* ALCOHOL_BRg*  BMIC_BR2g* CVD_BR12g* HT_BR12g* DIAB_BR12g* HYPERLIPIDEMIA_BRg* SRH_BRg* PHYSICAL_ACTIVITY_MET  if sample_final==1 , a0(0) a1(1) m(0) yreg(cox) mreg(linear) 
  
}

capture log close


capture log close

capture log using "E:\WHI_EPIGENETICCLOCK_DEP\OUTPUT\TABLE6.smcl",replace


////////////////////////TABLE 6: Causal mediation analysis for change in depression, antidepressant use as well as depression and/or antidepressant use as mediators and/or moderators of the relationship between estimates of epigenetic age acceleration and all-cause mortality risks ///////




 
 
capture drop AGE_BRg* 
tab AGE_BR,generate(AGE_BRg)


capture drop RACEBRg*
tab RACEBR, generate(RACEBRg)

capture drop ETHNICITYg*
tab ETHNICITY, generate(ETHNICITYg)

capture drop MARITAL_BRg*
tab MARITAL_BR, generate(MARITAL_BRg)

capture drop EDUC_BRg*
tab EDUC_BR, generate(EDUC_BRg)

capture drop INCOME_BRg*
tab INCOME_BR, generate(INCOME_BRg)

capture drop SMOKING_BRg*
tab SMOKING_BR, generate(SMOKING_BRg)
 
capture drop ALCOHOL_BRg*
tab ALCOHOL_BR, generate(ALCOHOL_BRg)

capture drop BMIC_BR2g*
tab BMIC_BR2, generate(BMIC_BR2g)

capture drop  CVD_BR12g*
tab CVD_BR12, generate(CVD_BR12g)

capture drop HT_BR12g*
tab HT_BR12, generate(HT_BR12g)

capture drop DIAB_BR12g*
tab DIAB_BR12, generate(DIAB_BR12g)

capture drop HYPERLIPIDEMIA_BRg*
tab HYPERLIPIDEMIA_BR, generate(HYPERLIPIDEMIA_BRg)

capture drop SRH_BRg*
tab SRH_BR, generate(SRH_BRg)


capture drop zIEAA zEEAA zAgeAccelPheno zAgeAccelGrimAge
foreach x of varlist IEAA EEAA AgeAccelPheno AgeAccelGrimAge {
	egen z`x'=std(`x') if sample_final==1
}

*************************************MED4WAY******************************************


**MODEL 1**

foreach x of varlist zIEAA zEEAA zAgeAccelPheno zAgeAccelGrimAge {
mi estimate, cmdok esampvaryok: med4way `x' DEPRESSION_BASELINE_BR     if sample_final==1 , a0(0) a1(1) m(0) yreg(cox) mreg(logistic) 
}


foreach x of varlist zIEAA zEEAA zAgeAccelPheno zAgeAccelGrimAge {
mi estimate, cmdok esampvaryok: med4way `x' ANTIDEPRESSANT_BR     if sample_final==1 , a0(0) a1(1) m(0) yreg(cox) mreg(logistic) 
}


foreach x of varlist zIEAA zEEAA zAgeAccelPheno zAgeAccelGrimAge {
mi estimate, cmdok esampvaryok: med4way  `x' DEP_ANTIDEP_DICH     if sample_final==1 , a0(0) a1(1) m(0) yreg(cox) mreg(logistic) 
  
}


**MODEL 2**

foreach x of varlist zIEAA zEEAA zAgeAccelPheno zAgeAccelGrimAge {
mi estimate, cmdok esampvaryok: med4way `x' DEPRESSION_BASELINE_BR  AGE_BRg* RACEBRg* ETHNICITYg* MARITAL_BRg* EDUC_BRg* INCOME_BRg*   if sample_final==1 , a0(0) a1(1) m(0) yreg(cox) mreg(logistic) 
}


foreach x of varlist zIEAA zEEAA zAgeAccelPheno zAgeAccelGrimAge {
mi estimate, cmdok esampvaryok: med4way `x' ANTIDEPRESSANT_BR  AGE_BRg* RACEBRg* ETHNICITYg* MARITAL_BRg* EDUC_BRg* INCOME_BRg*   if sample_final==1 , a0(0) a1(1) m(0) yreg(cox) mreg(logistic) 
}


foreach x of varlist zIEAA zEEAA zAgeAccelPheno zAgeAccelGrimAge {
mi estimate, cmdok esampvaryok: med4way `x' DEP_ANTIDEP_DICH  AGE_BRg* RACEBRg* ETHNICITYg* MARITAL_BRg* EDUC_BRg* INCOME_BRg*   if sample_final==1 , a0(0) a1(1) m(0) yreg(cox) mreg(logistic) 
  
}


**MODEL 3**

foreach x of varlist zIEAA zEEAA zAgeAccelPheno zAgeAccelGrimAge {
mi estimate, cmdok esampvaryok: med4way `x' DEPRESSION_BASELINE_BR  AGE_BRg* RACEBRg* ETHNICITYg* MARITAL_BRg* EDUC_BRg* INCOME_BRg* SMOKING_BRg* ALCOHOL_BRg*  BMIC_BR2g* CVD_BR12g* HT_BR12g* DIAB_BR12g* HYPERLIPIDEMIA_BRg* SRH_BRg* PHYSICAL_ACTIVITY_MET  if sample_final==1 , a0(0) a1(1) m(0) yreg(cox) mreg(logistic) 
}


foreach x of varlist zIEAA zEEAA zAgeAccelPheno zAgeAccelGrimAge {
mi estimate, cmdok esampvaryok: med4way `x' ANTIDEPRESSANT_BR  AGE_BRg* RACEBRg* ETHNICITYg* MARITAL_BRg* EDUC_BRg* INCOME_BRg* SMOKING_BRg* ALCOHOL_BRg*  BMIC_BR2g* CVD_BR12g* HT_BR12g* DIAB_BR12g* HYPERLIPIDEMIA_BRg* SRH_BRg* PHYSICAL_ACTIVITY_MET  if sample_final==1 , a0(0) a1(1) m(0) yreg(cox) mreg(logistic) 
}


foreach x of varlist zIEAA zEEAA zAgeAccelPheno zAgeAccelGrimAge {
mi estimate, cmdok esampvaryok: med4way `x' DEP_ANTIDEP_DICH  AGE_BRg* RACEBRg* ETHNICITYg* MARITAL_BRg* EDUC_BRg* INCOME_BRg* SMOKING_BRg* ALCOHOL_BRg*  BMIC_BR2g* CVD_BR12g* HT_BR12g* DIAB_BR12g* HYPERLIPIDEMIA_BRg* SRH_BRg* PHYSICAL_ACTIVITY_MET  if sample_final==1 , a0(0) a1(1) m(0) yreg(cox) mreg(logistic) 
  
}

capture log close

