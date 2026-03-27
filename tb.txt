import re
import sys
import os
print(sys.argv[1])
tb_path=re.findall(r'([/][A-Za-z0-9_/.-]+[/])?',sys.argv[1])[0]
file_name=re.findall(r'(\w+)[.]v',sys.argv[1])[0]
print(tb_path)
print(file_name)
with open(sys.argv[1],'r') as f1:
	r=f1.read();
	inputs=re.findall(r'input [[]\d+[:]\d+[]][A-Za-z0-9,_]+|input [a-zA-Z0-9,_]+',r)
	outputs=re.findall(r'output reg [[]\d+[:]\d+[]][A-Za-z0-9,_]+|output reg [A-Za-z0-9,_]+|output [[]\d+[:]\d+[]][A-Za-z0-9,_]+|output [a-zA-Z0-9_]+',r)
	inouts=re.findall(r'inout [[]\d+[:]\d+[]][A-Za-z0-9,_]+|inout [a-zA-Z0-9,_]+',r)
	module_name=re.findall(r'module (\w+)',r)[0]
	print('module name=',module_name)
	for i in range(len(inputs)):
		inputs[i]=inputs[i].replace('input','reg') #converted to reg inputs.
	in_vars=inputs+[]
	for i in range(len(inputs)):
		in_vars[i]=inputs[i].split('reg ')[1]
	for i in range(len(outputs)):
		l=outputs[i].find('reg')
		if(l!=-1):
			outputs[i]=outputs[i].split('reg')[1]
			outputs[i]='wire'+outputs[i] #converted to wire outputs
		else:
			outputs[i]=outputs[i].replace('output','wire') #converted to wire outputs
	out_vars=outputs+[]
	for i in range(len(out_vars)):
		out_vars[i]=out_vars[i].split('wire ')[1]
	for i in range(len(inouts)):
		inouts[i]=inouts[i].replace('inout','wire') #converted to wire inouts.
	inout_vars=inouts+[]
	for i in range(len(inouts)):
		inout_vars[i]=inout_vars[i].split('wire ')[1]
	##print(in_vars)
	maxs=[inputs,outputs,inouts] #list of converted inputs.
	for i in maxs:
		for j in range(len(i)):
			for k in range(len(i[j])):
				if(i[j][len(i[j])-1]==','): #removing , at the end of the vars.
					i[j]=i[j][0:len(i[j])-1];
	print(maxs)
	mins=[in_vars,out_vars,inout_vars] #list of variables need to remove , and vectors.
	print(mins)
	l=[]
	for i in mins:
		for j in range(len(i)):
			k=i[j].split(',') #removing , comas.
			l.extend(k)
			if(l[len(l)-1]==""):
				l.pop()
	for i in range(len(l)):
		k=l[i].find(']')
		if(k!=-1):
			l[i]=l[i].split(']')[1] #removing [] vectors. for instatiation.
	print('list=',l)
tb_name=tb_path+'tb_'+file_name+'.v'
c_flag=0;r_flag=0;
with open(tb_name,'w+') as f2:
	f2.writelines(['`include "',sys.argv[1],'"\n'])
	f2.writelines(['module ','tb_',module_name,';\n'])
	for i in maxs:
		for j in range(len(i)):
			f2.writelines([i[j],';\n']) #variable declaration.
	f2.writelines([module_name,' x1(']) #module instatiation.
	for i in range(len(l)):
		if(l[i]=='clk'):
			c_flag=1;
		if(l[i]=='rst'):
			r_flag=1;
		if(i==len(l)-1):
			f2.writelines(['.',l[i],'(',l[i],'));\n'])
		else:
			f2.writelines(['.',l[i],'(',l[i],'),'])
	if(c_flag==1):
		f2.write('always #5 clk=~clk;\n')
	f2.write('initial begin\n')
	f2.write('//please enter input vectors here....\n')
	if(c_flag==1):
		f2.write('clk=0;\n')
	if(r_flag==1):
		f2.write('rst=1;\n#5 rst=0\n #5 rst=1;\n\n\n')
	f2.write('#300;\n$finish;\n')
	f2.write('end\n')
	f2.write('initial begin\n')
	f2.writelines(['$recordfile("',file_name,'_waves.trn");\n'])
	f2.write("$recordvars();\n")
	f2.write('end\n')
	f2.write('endmodule')

