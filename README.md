import csv 
import unicodedata as uni
import sqlite3

labels=[]
filename=input("Enter name of the file")
text=[]
global tot_count
 
def read(file,text):         
    """reading the data"""
    with open(file,'r') as f:
        r=csv.reader(f)
        for row in r:
            for item in row:
                text.append(item)
    return text

def select_():
    '''reading data from table'''
    conn=sqlite3.connect('hackathon.db')
    c=conn.cursor()    
    jobs=[]
    for row in c.execute("SELECT DESCRIPTION FROM Jjob_indeed"):
        jobs.append(row)
    c.close()
    skills=[]
    for row in c.execute("SELECT SKILLS FROM linkedin_skills"):
        skills.append(row)
    c.close()
    conn.close()
    return jobs,skills
def calc(job,text):
    '''counting number of matching skills in the given resume'''
    count=0
    for uni in job:
        if uni in text:
            count=count+1
    return count
def cleanup(jobs,skills,txt):
    '''clean up of each job and skills'''
    for i in range(len(jobs)):
        jobs[i]=uni.normalize('NFKD',"".join(jobs[i]).encode('ascii','ignore'))
    #skills in 1 cell
    skills[0]=uni.normalize('NFKD',"".join(skills[0]).encode('ascii','ignore'))
    skills[1]=(skills[0].split(','))
    skill_list=[item.lower() for item in txt]
    jobss=[item.lower() for item in jobs]
    job_list=[]
    for job in jobss:
            job_list.append(job.split(' '))
    return skill_list,text,job_list

def accuracy(skill_list,text,job_list):
    """calc match accuracy"""
    match=[]
    for job in job_list:
        x=jobs_calc(job,text)
        y=jobs_calc(skill_list,job)
        match.append(float(y)/(x+1)*100)
    return match


def table(match):
    '''updating table in the sqlite data base'''
    conn =sqlite3.connect('hackathon.db')
    sql="UPDATE jobs_indeed SET ACCURACY= ? WHERE ID=?"
    for i in range(len(match)):
        cur=conn.cursor()
        cur.execute(sql,match[i],i+1)
        cur.close()
    conn.commit()
    conn.close()

def main():
    txt=read(filename,labels)
    jobs,skills=select_()
    skill_list,text,job_list=cleanup(jobs,skills,txt)
    skill=calc(skill_list,text)

    match=accuracy(skill_list,text,job_list)
    table(match)
