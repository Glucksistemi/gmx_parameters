# Preparation

1. Structure were cleaned using PyMol:
   1. `remove not polymer` (removing source water, ligands and ions, everything not in polymer chain)
   2. `remove not elem H` (removing source hydrogens)
2. run variables were defined (can be found in manuscript itself)

# GROMACS commands list

Following commands were runned automaticaly for every structure in set.
All runs were performed in separate folders containing set of initial `.mdp` files (provided [here](config_files)).
.pdb file containing structure were saved in work directory as protein.pdb

1. `gmx pdb2gmx -f protein.pdb -o conf.gro -water tip3p -ignh -missing -quiet`
    * input: 1
2. `gmx editconf -f conf.gro -o box.gro -c -d 1.0 -bt cubic -quiet`
3. `gmx solvate -cp box.gro -cs tip3p.gro -o box_s.gro -p topol.top -quiet`
4. `gmx grompp -f /root/gmxrunner/data/ions.mdp -c box_s.gro -p topol.top -o ions.tpr -maxwarn 2 -quiet`
5. `gmx genion -s ions.tpr -o box_s_i.gro -p topol.top -pname SOD -nname CLA -neutral -quiet`
    * input: 13
6. `gmx grompp -f /root/gmxrunner/data/em.mdp -c box_s_i.gro -p topol.top -o em.tpr -maxwarn 2 -quiet`
7. `gmx mdrun -deffnm em -nb cpu -pme cpu -nt 4 -quiet`
9. `gmx grompp -f /root/gmxrunner/data/heat.mdp -c em.gro -r em.gro -p topol.top -o heat.tpr -maxwarn 2 -quiet`
10. `gmx mdrun -deffnm heat -gpu_id 0 -nt 50 -quiet`
11. `gmx grompp -f /root/gmxrunner/data/md.mdp -c heat.gro -t heat.cpt -p topol.top -o md.tpr -maxwarn 2 -quiet`
12. `gmx mdrun -deffnm md -gpu_id 0 -nt 50 -quiet`

# Energy calculation
1. for every structure were created .ndx file containing lists of atoms for every PTM and every processed surround
2. for every given surround were performed commands:
   1. `gmx grompp -f md_e{postfix}_{group}.mdp -c heat.gro -t heat.cpt -n {file_name} -p topol.top -o md_e{postfix}_{group}.tpr -maxwarn 2 -quiet`
   2. `gmx mdrun -rerun md.xtc -deffnm md_e{postfix}_{group} -quiet`
parameters in `{}` were iterated over groups for surround and specific modification