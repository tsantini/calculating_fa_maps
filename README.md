### On matlab

1) Add `open_nii_TS` folder to your path
https://www.dropbox.com/t/EG0et4tnHiIz5DBe

2) modify the `open_nii_TS.m` according to the intructions inside the file

3) run this piece of code, 

```
files = dir([path_to_files filesep 'EPICS*']);

for i=1:length(files)
    clearvars -except files i
    if files(i).name(1)=='E' || files(i).name(1)=='1' || files(i).name(1)=='i'
        load([files(i).folder filesep files(i).name])
        variables = who;
        for j=1:length(variables)
            try
            if sum(eval([variables{j} ' ~= real(' variables{j} ')']),"all") %check if the variables is complex
                disp(variables{j})
                pat = ("1p"|"0p") + digitsPattern(1);
                text = extract(files(i).name,pat);
                size_image = eval(['size(' variables{j} ')'])
                resolution = str2double([text{1}(1) '.' text{1}(3)]);
                eval(sprintf("open_nii_TS(abs(flip(imrotate(%s,-90),1)),3,'%s',[%f %f %f], [%f %f %f])",variables{j}, erase(files(i).name,'.mat'), resolution, resolution, 3, round(size_image(1)/2), round(size_image(1)/2), 1 ))
            end
            catch ME
                disp(ME)
            end
        end
    end
end
```

### In the terminal

4) copy the nifti files it generated to a separate folder, for exemple, I'm keeping them at `220724-fa_from_mat_files`

5) run the bash script inside the `220724-fa_from_mat_file` called `compute_DTI.sh` using the tsantini/image_processing docker conteiner:

```
#!/bin/bash

for files in *.nii.gz; do
	[ ! -d ${files%.nii.gz} ] && mkdir ${files%.nii.gz}
	cd ${files%.nii.gz}
	[ ! -d ${files%.nii.gz}_tensor.mif ] && dwi2tensor ../$files -fslgrad ../dwi.bvec ../dwi.bval ${files%.nii.gz}_tensor.mif
	[ ! -d ${files%.nii.gz}_ev.nii.gz ] && tensor2metric  ${files%.nii.gz}_tensor.mif -adc ${files%.nii.gz}_adc.nii.gz -fa ${files%.nii.gz}_fa.nii.gz -ad ${files%.nii.gz}_ad.nii.gz -rd ${files%.nii.gz}_rd.nii.gz -vector ${files%.nii.gz}_ev.nii.gz
    cd -
done
```

6) your fa maps will be in a folder with the same name as the nifti file, gg
