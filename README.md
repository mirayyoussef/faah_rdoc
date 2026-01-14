%% ================================================================
%% MATLAB Analysis Script — CONSOLIDATED (No Figures)
%% ================================================================

%% -------------------------------
%% 1. Import Data
cd('C:\Users\peter\Documents\GRANTS\FAAH-RDOC\CIHR\prelim_analyses')
data = readtable('C:\Users\peter\Documents\GRANTS\FAAH-RDOC\CIHR\prelim_analyses\data\ALL CURB W NP.xlsx');

%% -------------------------------
%% 2. Recode Group IDs
data.scr_group(data.scr_group_id==1)  = {'HC'};
data.scr_group(data.scr_group_id==2)  = {'AUD'};
data.scr_group(data.scr_group_id==3)  = {'CUD'};
data.scr_group(data.scr_group_id==4)  = {'PTSD'};
data.scr_group(data.scr_group_id==6)  = {'SAD'};
data.scr_group(data.scr_group_id==9)  = {'NOUD'};
data.scr_group(data.scr_group_id==10) = {'YOUD'};

data.group = categorical(data.scr_group_id);

%% -------------------------------
%% 3. PET ROI Names
pet_roinames = data.Properties.VariableNames(26:80)';
pet_roinames_short = {'rmpfc1','rtemp2','rthal3','lmpfc4','ltemp5','lthal6','insula11','lacing12','racing13','vsr15','vsl16','sn17','aqu18','cereb19','amygdala20','rdlpfc21','ldlpfc22','rorbfr23','lorbfr24','b2525','hippo26','occ27','inf_parietal28','vstri31','rpredca39','rpostca40','rpredpu41','rpostpu42','lpredca43','lpostca44','lpredpu45','lpostpu46','predca50','postca51','predpu52','postpu53','rast54','last55','rdcaudate56','ldcaudate57','rdput58','ldput59','dcaudate60','dput61','dstri62','ast63','lst64','smst65','full_stri66','mpfc67','pfc68','temporal69','pftemporal70','thalamus71','cingulate72'};

%% -------------------------------
%% 4. Derived Variables
data.np1_igt_deckc_vs_d_raw = data.np1_igt_deckc_raw - data.np1_igt_deckd_raw;
data.gmp_param = data.PrevPost_Subgroup > 2;

%% -------------------------------
%% 5. Data Cleaning
data.np1_dd_small_k(data.np1_dd_small_k < 0) = NaN;
data.np1_dd_large_k(data.np1_dd_large_k < 0) = NaN;

data.pet1curb_curblk3_rpostca40 = [];
data.pet1curb_curblk3_lpostca44 = [];
data.pet1curb_curblk3_postca51 = [];

%% -------------------------------
%% 6. PET–PET Correlations
[r_pet, p_pet] = corr(data{:,26:80}, 'rows','pairwise');

%% -------------------------------
%% 7. Cognitive Correlation Matrices
[r_cog1,p_cog1] = corr([ ...
    data.np1_cpt_impulshit_rt_tscore, ...
    data.np1_dd_geomean_k, ...
    data.np1_igt_nettotal_raw * -1, ...
    data.np1_sdr_30s_both_mm], 'rows','pairwise');
r_cog1(eye(4)==1)=0;

[r_cog2,p_cog2] = corr([ ...
    data.np1_sst_mean_ssrt, ...
    data.np1_sst_ssrt_sd, ...
    data.np1_trails_b, ...
    data.np1_cpt_impulshit_rt_tscore, ...
    data.np1_hvlt_total * -1, ...
    data.np1_sdr_30s_both_mm], 'rows','pairwise');
r_cog2(eye(6)==1)=0;

%% -------------------------------
%% 8. PET–Cognition Correlations (Screening)
[r_trails, p_trails] = corr(data.np1_trails_b, data{:,26:80}, 'rows','pairwise');
[r_digit,  p_digit]  = corr(data.np1_digit_forward, data{:,26:80}, 'rows','pairwise');
[r_hvlt,   p_hvlt]   = corr(data.np1_hvlt_total, data{:,26:80}, 'rows','pairwise');
[r_hvlt2,  p_hvlt2]  = corr(data.np1_hvlt_higher_of_2or3, data{:,26:80}, 'rows','pairwise');
[r_peg,    p_peg]    = corr(data.np1_peg_both, data{:,26:80}, 'rows','pairwise');

%% -------------------------------
%% 9. Key Impulsivity / Decision Measures
[r_cpt, p_cpt] = corr(data.np1_cpt_impulshit_rt_tscore, data{:,26:80}, 'rows','pairwise');
[r_dd,  p_dd]  = corr(data.np1_dd_large_k, data{:,26:80}, 'rows','pairwise');
[r_sst_crt, p_sst_crt] = corr(data.np1_sst_mean_crt, data{:,26:80}, 'rows','pairwise');
[r_sst_ssrt,p_sst_ssrt]= corr(data.np1_sst_mean_ssrt,data{:,26:80}, 'rows','pairwise');

[r_igt_cd, p_igt_cd] = corr(data.np1_igt_deckc_vs_d_raw, data{:,26:80}, 'rows','pairwise');
[r_igt_d,  p_igt_d]  = corr(data.np1_igt_deckd_raw, data{:,26:80}, 'rows','pairwise');
[r_igt_total,p_igt_total]=corr(data.np1_igt_total_money,data{:,26:80},'rows','pairwise');
[r_igt_net5,p_igt_net5]=corr(data.np1_igt_net5_raw,data{:,26:80},'rows','pairwise');

[r_sdr,p_sdr]=corr(data.np1_sdr_30s_both_mm,data{:,26:80},'rows','pairwise');

%% -------------------------------
%% 10. PET Correlation Summary
min_p_pet = min(p_pet(:));
max_r_pet = max(r_pet(:));
min_r_pet = min(r_pet(:));
pet_rois_pos = pet_roinames(r_pet > 0.2);
pet_rois_neg = pet_roinames(r_pet < -0.2);

%% -------------------------------
%% 11. PET → Cognition Linear Models (with covariates)
mdl_sdr  = fitlm(data,'np1_sdr_30s_both_mm ~ pet1curb_curblk3_rmpfc1 + scr1_sex + scr1_age + faah_genotype + gmp_param');
mdl_cpt  = fitlm(data,'np1_cpt_impulshit_rt_tscore ~ pet1curb_curblk3_hippo26 + scr1_sex + scr1_age + faah_genotype + gmp_param');
mdl_dd   = fitlm(data,'np1_dd_small_k ~ pet1curb_curblk3_racing13 + scr1_sex + scr1_age + faah_genotype + gmp_param');
mdl_igt1 = fitlm(data,'np1_igt_deckc_raw ~ pet1curb_curblk3_amygdala20 + scr1_sex + scr1_age + faah_genotype + gmp_param');
mdl_igt2 = fitlm(data,'np1_igt_deckc_raw ~ pet1curb_curblk3_rorbfr23 + scr1_sex + scr1_age + faah_genotype + gmp_param');

%% -------------------------------
%% 12. Residual-Based (Partial) Associations
mdl_sdr_cov = fitlm(data,'np1_sdr_30s_both_mm ~ scr1_sex + scr1_age + faah_genotype + gmp_param');
mdl_dd_cov  = fitlm(data,'np1_dd_small_k ~ scr1_sex + scr1_age + faah_genotype + gmp_param');
mdl_igt_cov = fitlm(data,'np1_igt_deckc_raw ~ scr1_sex + scr1_age + faah_genotype + gmp_param');
mdl_amyg_cov= fitlm(data,'pet1curb_curblk3_amygdala20 ~ scr1_sex + scr1_age + faah_genotype + gmp_param');

[r_resid_igt, p_resid_igt] = corr(mdl_amyg_cov.Residuals.Raw, mdl_igt_cov.Residuals.Raw,'rows','pairwise');

%% -------------------------------
%% 13. Whole-Brain PET Screening (Covariates Only)
for i=1:42
    mdl = fitlm(data,[pet_roinames{i} ' ~ scr1_sex + scr1_age + gmp_param']);
    tstat_cov(i) = mdl.Coefficients.tStat(2);
    pval_cov(i)  = mdl.Coefficients.pValue(2);
end
cohen_d_cov = tstat_cov ./ sqrt(height(data));
pval_cov_fdr = bhfdr(pval_cov);

%% -------------------------------
%% 14. PET ROI ~ Group + Covariates (Whole-Brain)
for i=1:42
    mdl = fitlm(data,[pet_roinames{i} ' ~ scr1_sex + scr1_age + group + gmp_param']);
    a = anova(mdl);
    F_group(i) = a.F(4);
    p_group(i) = a.pValue(4);
end
p_group_fdr = bhfdr(p_group);

%% -------------------------------
%% 15. Cognitive Outcomes ~ Group + Covariates
mdl_igt_grp = fitlm(data,'np1_igt_nettotal_tscore ~ scr1_sex + scr1_age + group');
mdl_sdr_grp = fitlm(data,'np1_sdr_30s_both_mm ~ scr1_sex + scr1_age + group');
mdl_cpt_grp = fitlm(data,'np1_cpt_impulshit_rt_tscore ~ scr1_sex + scr1_age + group');
mdl_dd_grp  = fitlm(data,'np1_dd_small_k ~ scr1_sex + scr1_age + group');
mdl_trl_grp = fitlm(data,'np1_trails_b ~ scr1_sex + scr1_age + group');

%% -------------------------------
%% 16. FAAH Genotype Effects
anova1(data.pet1curb_curblk3_amygdala20, data.faah_genotype);
anova1(data.np1_cpt_impulshit_rt_tscore, data.faah_genotype);

%% -------------------------------
%% 17. Delay Discounting Grouping Analysis
data.np1_dd_small_k_group = discretize(data.np1_dd_small_k,[0 0.05 0.1 0.2 inf]);
anova1(data.pet1curb_curblk3_racing13, data.np1_dd_small_k_group);

%% -------------------------------
%% 18. IGT Deck Correlation Matrix
[r_igt_decks,p_igt_decks] = corr([ ...
    data.np1_igt_deckc_raw - data.np1_igt_deckd_raw, ...
    data.np1_igt_decka_raw, ...
    data.np1_igt_deckb_raw, ...
    data.np1_igt_deckc_raw, ...
    data.np1_igt_deckd_raw, ...
    data.np1_igt_total_money], 'rows','pairwise');

%% -------------------------------
%% 19. Descriptive Crosstabs
tbl_group_sex = crosstab(data.scr_group_id, data.scr1_sex);
tbl_group_gmp = crosstab(data.scr_group_id, data.gmp_param);

%% ================================================================
%% END OF SCRIPT
%% ================================================================
