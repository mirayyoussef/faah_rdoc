%% ================================================================
%% MATLAB Analysis Script (No Figures) — REORGANIZED
%% ================================================================

%% -------------------------------
%% 1. Import Data
cd('C:\Users\peter\Documents\GRANTS\FAAH-RDOC\CIHR\prelim_analyses'); % Set working directory
data = readtable('C:\Users\peter\Documents\GRANTS\FAAH-RDOC\CIHR\prelim_analyses\data\ALL CURB W NP.xlsx'); % Load Excel data

%% -------------------------------
%% 2. Recode Group IDs to String Labels
data.scr_group(data.scr_group_id==1)  = {'HC'};
data.scr_group(data.scr_group_id==2)  = {'AUD'};
data.scr_group(data.scr_group_id==3)  = {'CUD'};
data.scr_group(data.scr_group_id==4)  = {'PTSD'};
data.scr_group(data.scr_group_id==6)  = {'SAD'};
data.scr_group(data.scr_group_id==9)  = {'NOUD'};
data.scr_group(data.scr_group_id==10) = {'YOUD'};

%% -------------------------------
%% 3. Convert Group ID to Categorical
data.group = categorical(data.scr_group_id);

%% -------------------------------
%% 4. Define PET ROI Names
pet_roinames = data.Properties.VariableNames(26:80)'; % Full variable names for PET ROIs
pet_roinames_short = {'rmpfc1','rtemp2','rthal3','lmpfc4','ltemp5','lthal6','insula11','lacing12','racing13','vsr15','vsl16','sn17','aqu18','cereb19','amygdala20','rdlpfc21','ldlpfc22','rorbfr23','lorbfr24','b2525','hippo26','occ27','inf_parietal28','vstri31','rpredca39','rpostca40','rpredpu41','rpostpu42','lpredca43','lpostca44','lpredpu45','lpostpu46','predca50','postca51','predpu52','postpu53','rast54','last55','rdcaudate56','ldcaudate57','rdput58','ldput59','dcaudate60','dput61','dstri62','ast63','lst64','smst65','full_stri66','mpfc67','pfc68','temporal69','pftemporal70','thalamus71','cingulate72'};

%% -------------------------------
%% 5. Derived Variables
data.np1_igt_deckc_vs_d_raw = data.np1_igt_deckc_raw - data.np1_igt_deckd_raw; % Deck C minus D
data.gmp_param = data.PrevPost_Subgroup > 2; % Binary subgroup variable

%% -------------------------------
%% 6. Handle Invalid Delay Discounting Values
data.np1_dd_small_k(data.np1_dd_small_k < 0) = NaN;
data.np1_dd_large_k(data.np1_dd_large_k < 0) = NaN;

%% -------------------------------
%% 7. Remove Problematic PET Columns
data.pet1curb_curblk3_rpostca40 = []; 
data.pet1curb_curblk3_lpostca44 = []; 
data.pet1curb_curblk3_postca51 = [];

%% -------------------------------
%% 8. PET ROI Correlations
[r_pet, p_pet] = corr(data{:,26:80}, 'rows', 'pairwise'); % Pairwise correlation for all PET ROIs

%% -------------------------------
%% 9. Cognitive Correlations (Broad Overview)
% CPT, DD, IGT, SDR
[r_cog1,p_cog1] = corr([data.np1_cpt_impulshit_rt_tscore, data.np1_dd_geomean_k, data.np1_igt_nettotal_raw*-1, data.np1_sdr_30s_both_mm], 'rows', 'pairwise'); 
r_cog1(eye(4)==1) = 0; % Remove diagonal

% SST, Trails, HVLT, CPT, SDR
[r_cog2,p_cog2] = corr([data.np1_sst_mean_ssrt, data.np1_sst_ssrt_sd, data.np1_trails_b, data.np1_cpt_impulshit_rt_tscore, data.np1_hvlt_total*-1, data.np1_sdr_30s_both_mm], 'rows', 'pairwise'); 
r_cog2(eye(6)==1) = 0;

%% -------------------------------
%% 10. Classical Controls (general cognitive/motor measures)
% Executive function / attention
[r_trails, p_trails] = corr(data.np1_trails_b, data{:,26:80}, 'rows', 'pairwise'); % Trails B vs PET ROIs
[r_digit, p_digit] = corr(data.np1_digit_forward, data{:,26:80}, 'rows', 'pairwise'); % Digit span forward vs PET ROIs

% Memory measures
[r_hvlt, p_hvlt] = corr(data.np1_hvlt_total, data{:,26:80}, 'rows', 'pairwise'); % HVLT total vs PET ROIs
[r_hvlt2, p_hvlt2] = corr(data.np1_hvlt_higher_of_2or3, data{:,26:80}, 'rows', 'pairwise'); % HVLT highest trials vs PET ROIs

% Motor control / dexterity
[r_peg, p_peg] = corr(data.np1_peg_both, data{:,26:80}, 'rows', 'pairwise'); % Pegboard vs PET ROIs

%% -------------------------------
%% 11. Key Cognitive Measures (Impulsivity & Decision-Making)
% Attention / inhibition
[r_cpt, p_cpt] = corr(data.np1_cpt_impulshit_rt_tscore, data{:,26:80}, 'rows', 'pairwise'); % CPT impulsivity vs PET ROIs

% Delay discounting
[r_dd, p_dd] = corr(data.np1_dd_large_k, data{:,26:80}, 'rows', 'pairwise'); % Delay discounting k vs PET ROIs

% Response inhibition (SST)
[r_sst_crt, p_sst_crt] = corr(data.np1_sst_mean_crt, data{:,26:80}, 'rows', 'pairwise'); % SST correct RT vs PET ROIs
[r_sst_ssrt, p_sst_ssrt] = corr(data.np1_sst_mean_ssrt, data{:,26:80}, 'rows', 'pairwise'); % SST SSRT vs PET ROIs

% Iowa Gambling Task (IGT)
[r_igt_cd, p_igt_cd] = corr(data.np1_igt_deckc_raw - data.np1_igt_deckd_raw, data{:,26:80}, 'rows', 'pairwise'); % Deck C-D vs PET ROIs
[r_igt_d, p_igt_d] = corr(data.np1_igt_deckd_raw, data{:,26:80}, 'rows', 'pairwise'); % Deck D vs PET ROIs
[r_igt_total, p_igt_total] = corr(data.np1_igt_total_money, data{:,26:80}, 'rows', 'pairwise'); % IGT total vs PET ROIs
[r_igt_net5, p_igt_net5] = corr(data.np1_igt_net5_raw, data{:,26:80}, 'rows', 'pairwise'); % IGT net5 vs PET ROIs

% Spatial / working memory (SDR)
[r_sdr, p_sdr] = corr(data.np1_sdr_30s_both_mm, data{:,26:80}, 'rows', 'pairwise'); % SDR vs PET ROIs

%% -------------------------------
%% 12. Correlation Summary for PET
min_p = min(p_pet(:)); % Minimum p-value
max_r = max(r_pet(:));  % Maximum correlation
min_r = min(r_pet(:));  % Minimum correlation
pet_roinames_neg = pet_roinames(r_pet<-0.2); % PET ROIs with r<-0.2
pet_roinames_pos = pet_roinames(r_pet>0.2);  % PET ROIs with r>0.2

%% -------------------------------
%% 13. Linear Regression Controlling for Confounders
mdl_sdr  = fitlm(data,'np1_sdr_30s_both_mm ~ pet1curb_curblk3_rmpfc1 + scr1_sex + scr1_age + faah_genotype + gmp_param');
mdl_cpt  = fitlm(data,'np1_cpt_impulshit_rt_val ~ pet1curb_curblk3_hippo26 + scr1_sex + scr1_age + faah_genotype + gmp_param');
mdl_dd   = fitlm(data,'np1_dd_small_k ~ pet1curb_curblk3_racing13 + scr1_sex + scr1_age + faah_genotype + gmp_param');
mdl_igt1 = fitlm(data,'np1_igt_deckc_raw ~ pet1curb_curblk3_amygdala20 + scr1_sex + scr1_age + faah_genotype + gmp_param');
mdl_igt2 = fitlm(data,'np1_igt_deckc_raw ~ pet1curb_curblk3_rorbfr23 + scr1_sex + scr1_age + faah_genotype + gmp_param');

%% -------------------------------
%% 14. Linear Models for PET ROIs vs Covariates Only
mdl_vstri = fitlm(data,'pet1curb_curblk3_vstri31 ~ scr1_sex + scr1_age + faah_genotype + gmp_param');
mdl_hippo = fitlm(data,'pet1curb_curblk3_hippo26 ~ scr1_sex + scr1_age + faah_genotype + gmp_param');
mdl_lacing = fitlm(data,'pet1curb_curblk3_lacing12 ~ scr1_sex + scr1_age + faah_genotype + gmp_param');
mdl_rorb  = fitlm(data,'pet1curb_curblk3_rorbfr23 ~ scr1_sex + scr1_age + faah_genotype + gmp_param');
mdl_amyg  = fitlm(data,'pet1curb_curblk3_amygdala20 ~ scr1_sex + scr1_age + faah_genotype + gmp_param');

%% -------------------------------
%% 15. Linear Models for PET ROIs vs Group + Covariates
mdl_rorb_grp  = fitlm(data,'pet1curb_curblk3_rorbfr23 ~ scr1_sex + scr1_age + group + gmp_param');
mdl_amyg_grp  = fitlm(data,'pet1curb_curblk3_amygdala20 ~ scr1_sex + scr1_age + group + gmp_param');
mdl_vstri_grp = fitlm(data,'pet1curb_curblk3_vstri31 ~ scr1_sex + scr1_age + group + gmp_param');

% Loop over all 42 ROIs
for i = 1:42
    mdl = fitlm(data,strcat(pet_roinames{i},' ~ scr1_sex + scr1_age + group + gmp_param'));
    anova_mdl{i} = anova(mdl); %#ok<SAGROW>
end

%% -------------------------------
%% 16. Cognitive Outcomes vs Group + Covariates
mdl_igt  = fitlm(data,'np1_igt_nettotal_tscore ~ scr1_sex + scr1_age + group'); anova_igt = anova(mdl_igt);
mdl_sdr  = fitlm(data,'np1_sdr_30s_both_mm ~ scr1_sex + scr1_age + group'); anova_sdr = anova(mdl_sdr);
mdl_cpt  = fitlm(data,'np1_cpt_impulshit_rt_tscore ~ scr1_sex + scr1_age + group'); anova_cpt = anova(mdl_cpt);
mdl_dd   = fitlm(data,'np1_dd_small_k ~ scr1_sex + scr1_age + group'); anova_dd = anova(mdl_dd);
mdl_trails = fitlm(data,'np1_trails_b ~ scr1_sex + scr1_age + group'); anova_trails = anova(mdl_trails);

%% -------------------------------
%% 17. ANOVAs by Group
[p_amyg, tbl_amyg, stats_amyg] = anova1(data.pet1curb_curblk3_amygdala20, data.scr_group_id);
[p_igt_cd, tbl_igt_cd, stats_igt_cd] = anova1(data.np1_igt_deckc_vs_d_raw, data.scr_group_id);

% Remove group 6 if necessary
data(data.scr_group_id==6,:) = [];
anova1(data.np1_igt_deckc_raw, data.scr_group_id);
anova1(data.np1_igt_total_money, data.scr_group_id);
anova1(data.pet1curb_curblk3_rmpfc1, data.scr_group_id);
anova1(data.pet1curb_curblk3_dput61, data.scr_group_id);
anova1(data.np1_cpt_impulshit_rt_tscore, data.scr_group_id);

% Delay discounting grouping
data.np1_dd_small_k_group = data.np1_dd_small_k;
data.np1_dd_small_k_group(data.np1_dd_small_k < 0.05) = 0;
data.np1_dd_small_k_group(data.np1_dd_small_k < 0.1 & data.np1_dd_small_k > 0.05) = 1;
data.np1_dd_small_k_group(data.np1_dd_small_k < 0.2 & data.np1_dd_small_k > 0.1) = 2;
data.np1_dd_small_k_group(data.np1_dd_small_k > 0.2) = 3;
[p_dd_group, tbl_dd_group, stats_dd_group] = anova1(data.pet1curb_curblk3_racing13, data.np1_dd_small_k_group);

%% -------------------------------
%% 18. IGT Decks Correlations
[r_igt_decks, p_igt_decks] = corr([data.np1_igt_deckc_raw-data.np1_igt_deckd_raw, ...
                                   data.np1_igt_decka_raw, ...
                                   data.np1_igt_deckb_raw, ...
                                   data.np1_igt_deckc_raw, ...
                                   data.np1_igt_deckd_raw, ...
                                   data.np1_igt_total_money], 'rows', 'pairwise');

%% ================================================================
%% END OF SCRIPT
%% ================================================================
