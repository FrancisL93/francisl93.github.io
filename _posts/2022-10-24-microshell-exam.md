---
layout: post
title: Microshell_exam
date: 2022-10-24 13:16 -0400
categories: [Coding]
tags: [exam, microshell]
img_path: /assets/img/
image: /assets/img/coding.jpg
---
# After minishell... MICROSHELL!

For the complete code version of my microshell: [Microshell GitHub repo](url:https://github.com/FrancisL93/microshell_exam_test)

## No one asked, but here is my mind process to solve the exam:

### 1-First I start my main structure:
```c
int	main(int argc, char **argv, char **envp) {

}
```
**int argc:** number of parameters (1 if none, 2 if 1, etc...) \
**char \*\*argv:** every parameters stocked as argv[paremeters] \
**char \*\*envp:** environement variables stocked as envp[...] \
envp is needed to pass to execve() as requested in subject
### 2-Manage bad program utilization
```c
int	main(int argc, char **argv, char **envp) {
	if (argc ==1) { //if no argument provided -> do nothing & quit!
		return (1);
	}
}
```
### 3-Define a struct to easily use variables
```c
typedef struct s_vars
{
	int	fd[2]; //2 fds for when we will call pipe(vars->fd)
	int	*position; //If the command is the last or first of a sequence
	char	***cmds; //To place commands and their arguments in a "diagram"
} t_vars;

int	main(int argc, char **argv, char **envp) {
	t_vars vars; //locally declared -> malloc/free not needed!

	if (argc ==1) {
		return (1);
	}
}
```
### 4-Allocate the right memory to struct variables
```c
#include <string.h> // strcmp library
#include <stdlib.h> // malloc library

//count the number of commands... ex: "/bin/ls -lia ";" /bin/ls" -> = 2
int	count_commands(char **argv) {
	int count = 1;

	for (int j = 0; argv[j]; j++){
		if (!strcmp(argv[j], "|") || strcmp(argv[j], ";"))
			count++;
	}
	return (count);
}

int	main(int argc, char **argv, char **envp) {
	t_vars vars;

	if (argc ==1) {
		return (1);
	}
	vars.position = malloc(sizeof(int) * count_commands(argv));
	vars.cmds = malloc(sizeof(char **) * count_commands(argv));
	if (!vars.position || !vars.cmds) {
		return (1);
	}
}
```
### 5-Set the vars.cmds pointers to the right commands
```c
void	set_commands(t_vars *vars, char **argv) {

}

int	main(int argc, char **argv, char **envp) {
	t_vars vars;

	if (argc ==1) {
		return (1);
	}
	vars.position = malloc(sizeof(int) * count_commands(argv));
	vars.cmds = malloc(sizeof(char **) * count_commands(argv));
	if (!vars.position || !vars.cmds) {
		return (1);
	}
	set_commands(&vars, argv);
}
```
#### 5.1-Start with the first command (Ex: ./microshell "/bin/ls")
```c
void	set_commands(t_vars *vars, char **argv) {
	int cmd_list = 0;

	vars->cmds[0] = &argv[1];
}
```
#### 5.2-Create a simple loop without parsing (Ex: ./microshell "/bin/ls" "/bin/ls")
```c
void	set_commands(t_vars *vars, char **argv) {
	int cmd_list = 0;

	for (int i = 0; argv[i]; i++) {
		vars->cmds[cmd_list++] = &argv[++i];
	}
}
```
#### 5.3-Add filter to skip "|" & ";"
```c
void	set_commands(t_vars *vars, char **argv) {
	int cmd_list = 0;

	for (int i = 0; argv[i]; i++) {
		if (i == 0 || !strcmp(argv[i], "|") || !strcmp(argv[i], ";")) {
			vars->cmds[cmd_list++] = &argv[++i];
		}
	}
}
```
#### 5.4-Add loop to skip multiple ";"
```c
void	set_commands(t_vars *vars, char **argv) {
	int cmd_list = 0;

	for (int i = 0; argv[i]; i++) {
		if (i == 0 || !strcmp(argv[i], "|") || !strcmp(argv[i], ";")) {
			while (argv[i + 1] && !strcmp(argv[i + 1], ";"))
				i++;
			if (!argv[i + 1]) // important if command ends with ";"
				break ;
			vars->cmds[cmd_list++] = &argv[++i];
		}
	}
}
```
#### 5.5-Set commands positions to execute pipes
```c
#define FIRST	1
#define LAST	2

void	set_commands(t_vars *vars, char **argv) {
	int cmd_list = 0;

	for (int i = 0; argv[i]; i++) {
		if (i == 0 || !strcmp(argv[i], "|") || !strcmp(argv[i], ";")) {
			while (argv[i + 1] && !strcmp(argv[i + 1], ";"))
				i++;
			if (!argv[i + 1]) // important if command ends with ";"
				break ;
			vars->cmds[cmd_list++] = &argv[++i];
		}
		if (cmd_list == 1 || !strcmp(argv[i - 1], ";"))
			vars->position[cmd_list - 1] = FIRST; // define FIRST to 1
		if (!strcmp(argv[i - 1], ";"))
			vars->position[cmd_list - 2] = LAST // define LAST to 2
		//First is for when command is the first of a "|" sequence
		//Last is for when command is the last of a "|" sequence
		//If only one command it will be reset to LAST by 2nd if
	}
	vars->position[cmd_list - 1] = LAST;
	vars->cmds[cmd_list] = NULL;
}
```
### 6-Set stops to every command pointer
```c
void	set_stops(t_vars *vars) {
	for (int i = 0; vars->cmds[i]; i++) { // accessing /bin/ls -lia ...
		for (int j = 0; vars->cmds[i][j]; j++) { //accessing /bin/ls
			if (!strcmp(vars->cmds[i][j], "|") ||
			\!strcmp(vars->cmds[i][j], ";"))
				vars->cmds[i][j] = NULL;
		}
	}
}

int	main(int argc, char **argv, char **envp) {
	t_vars vars;

	if (argc ==1) {
		return (1);
	}
	vars.position = malloc(sizeof(int) * count_commands(argv));
	vars.cmds = malloc(sizeof(char **) * count_commands(argv));
	if (!vars.position || !vars.cmds) {
		return (1);
	}
	set_commands(&vars, argv);
	set_stops(&vars);
}
```
### 7-Starting the execution
```c
//needs to receive envp for execve
void	exe(t_vars *vars, char **envp) {

}
int	main(int argc, char **argv, char **envp) {
	t_vars vars;

	if (argc ==1) {
		return (1);
	}
	vars.position = malloc(sizeof(int) * count_commands(argv));
	vars.cmds = malloc(sizeof(char **) * count_commands(argv));
	if (!vars.position || !vars.cmds) {
		return (1);
	}
	set_commands(&vars, argv);
	set_stops(&vars);
	exe(&vars, envp);
}
```
#### 7.1-Declaring variables
```c
void	exe(t_vars *vars, char **envp) {
	pid_t pid ; // needed for executing commands in the children (fork())
	int ret = 0; // needed to skip fork () if cmd is "cd"
}
```
#### 7.2-Making a loop to execute all commands
```c
void	exe(t_vars *vars, char **envp) {
	pid_t pid ;
	int ret = 0;

	for (int i = 0; vars->cmds[i]; i++) {
		//execute every commands
	}
}
```
#### 7.3-Skipping execve if cmd is "cd"
```c
int		check_cd(t_vars *vars, int i) {

}

void	exe(t_vars *vars, char **envp) {
	pid_t pid ;
	int ret = 0;

	for (int i = 0; vars->cmds[i]; i++) {
		ret = check_cd(vars, i); // if check_cd() returns 1 we skip the rest
	}
}
```
#### 7.4-Building the check_cd() function
```c
int	ft_strlen(char *str) {
	int count = 0;
	for (int i = 0; str && str[i]; i++) {
		count++;
	}
	return (count);
}

int		check_cd(t_vars *vars, int i) {
	if (!strcmp(vars->cmds[i][0], "cd")) {
		//checks if first argument of the comment is built-in "cd"
		if (!vars->cmds[i][1] || vars->cmds[1][2]) {
			//checks if there is more than 1 argument pass to cd
			write(STDERR_FILENO, "error:cd: bad arguments\n", 25);
		}
		else if (chdir(vars->cmds[i][1]) != 0) {
			//checks if change directory function failed
			write(STDERR_FILENO, "error: cd: cannot change directory to ", 38);
			write(STDERR_FILENO, vars->cmds[i][1], ft_strlen(vars->cmds[i][1]));
			write(STDERR_FILENO, "\n", 1);
		}
		return (1); // return 1 in both cases to skip execve
	}
	return (0); // return 0 when the cmd is not cd
}
```
#### 7.5-Starting the execution
```c
void	exe(t_vars *vars, char **envp) {
	pid_t pid ;
	int ret = 0;

	for (int i = 0; vars->cmds[i]; i++) {
		ret = check_cd(vars, i); // if check_cd() returns 1 we skip the rest
		if (!ret && vars->position[i] != LAST)
			pipe(vars->fd); // create a communication between commands
		if (!ret)
			pid = fork(); // Creating a child process to execute command
		if (!ret && pid == 0) { // only child gets in the if
		
		}
	}
}
```
#### 7.6-Child process output management
```c
//fd[0] = read end of pipe, fd[1] = write end of pipe
//STDOUT_FILENO = 1, we dup2 to write into the pipe
void	set_pipe(t_vars *vars, int i) {
	if (vars->position[i] != last){ //if not last command, go through pipe
		dup2(vars->fd[1], STDOUT_FILENO);
		close(vars->fd[1]); // limit of 30 fd, so close every unused fd
		close(vars->fd[0]);
	}
}

void	exe(t_vars *vars, char **envp) {
	pid_t pid ;
	int ret = 0;

	for (int i = 0; vars->cmds[i]; i++) {
		ret = check_cd(vars, i); // if check_cd() returns 1 we skip the rest
		if (!ret && vars->position[i] != LAST)
			pipe(vars->fd); // create a communication between commands
		if (!ret)
			pid = fork(); // Creating a child process to execute command
		if (!ret && pid == 0) { // only child gets in the if statement
			set_pipe(vars, i);
		} 
	}
}
```
#### 7.7-Manage execve
```c
//1st argument of execve is the command (ex: /bin/ls)
//	which is related to pointer vars->cmds[i][0]
//2nd argument is the command and its arguments (ex: /bin/ls -l -i)
//  which is related to pointer vars->cmds[i], last argument needs to be NULL
//3rd argument is the environment
void	exe(t_vars *vars, char **envp) {
	pid_t pid ;
	int ret = 0;

	for (int i = 0; vars->cmds[i]; i++) {
		ret = check_cd(vars, i); // if check_cd() returns 1 we skip the rest
		if (!ret && vars->position[i] != LAST)
			pipe(vars->fd); // create a communication between commands
		if (!ret)
			pid = fork(); // Creating a child process to execute command
		if (!ret && pid == 0) { // only child gets in the if statement
			set_pipe(vars, i);
			execve(vars->cmds[i][0], vars->cmds[i], envp);
		} 
	}
}
```
#### 7.8-Manage execve error
```c
int	ft_strlen(char *str) {
	int count = 0;
	for (int i = 0; str && str[i]; i++) {
		count++;
	}
	return (count);
}

void	exe(t_vars *vars, char **envp) {
	pid_t pid ;
	int ret = 0;

	for (int i = 0; vars->cmds[i]; i++) {
		ret = check_cd(vars, i); // if check_cd() returns 1 we skip the rest
		if (!ret && vars->position[i] != LAST)
			pipe(vars->fd); // create a communication between commands
		if (!ret)
			pid = fork(); // Creating a child process to execute command
		if (!ret && pid == 0) { // only child gets in the if statement
			set_pipe(vars, i);
			execve(vars->cmds[i][0], vars->cmds[i], envp);
			write(STDERR_FILENO, "error: cannot execute ", 22);
			write(STDERR_FILENO, vars->cmds[i][0], ft_strlen(vars->cmds[i][0]));
			write(STDERR_FILENO, "\n", 1);
			exit(127);
		} 
	}
}
```
#### 7.8-Parent input management
```c
void	exe(t_vars *vars, char **envp) {
	pid_t pid ;
	int ret = 0;

	for (int i = 0; vars->cmds[i]; i++) {
		ret = check_cd(vars, i); // if check_cd() returns 1 we skip the rest
		if (!ret && vars->position[i] != LAST)
			pipe(vars->fd); // create a communication between commands
		if (!ret)
			pid = fork(); // Creating a child process to execute command
		if (!ret && pid == 0) { // only child gets in the if statement
			set_pipe(vars, i);
			execve(vars->cmds[i][0], vars->cmds[i], envp);
			write(STDERR_FILENO, "error: cannot execute ", 22);
			write(STDERR_FILENO, vars->cmds[i][0], ft_strlen(vars->cmds[i][0]));
			write(STDERR_FILENO, "\n", 1);
			exit(127);
		}
		if (!ret && vars->position[i] != LAST) {
			//STDIN_FILENO = 0, we dup2 to read from the pipe
			dup2(vars->fd[0], STDIN_FILENO);
			close(vars->fd[0]); // again, close all unused fds!
			close(vars->fd[1]);
		}
	}
}
```
#### 7.8-Parent input management
```c
//add int status to save exit status with waitpid() (wait() is illegal...)
void	exe(t_vars *vars, char **envp) {
	pid_t pid ;
	int ret = 0;
	int status;

	for (int i = 0; vars->cmds[i]; i++) {
		ret = check_cd(vars, i); // if check_cd() returns 1 we skip the rest
		if (!ret && vars->position[i] != LAST)
			pipe(vars->fd); // create a communication between commands
		if (!ret)
			pid = fork(); // Creating a child process to execute command
		if (!ret && pid == 0) { // only child gets in the if statement
			set_pipe(vars, i);
			execve(vars->cmds[i][0], vars->cmds[i], envp);
			write(STDERR_FILENO, "error: cannot execute ", 22);
			write(STDERR_FILENO, vars->cmds[i][0], ft_strlen(vars->cmds[i][0]));
			write(STDERR_FILENO, "\n", 1);
			exit(127);
		}
		if (!ret && vars->position[i] != LAST) {
			//STDIN_FILENO = 0, we dup2 to read from the pipe
			dup2(vars->fd[0], STDIN_FILENO);
			close(vars->fd[0]); // again, close all unused fds!
			close(vars->fd[1]);
		}
		if (!ret)
			waitpid(pid, &status, 0);
	}
}
```
#### 7.8-Close the program cleanly without leaks
```c
int	main(int argc, char **argv, char **envp) {
	t_vars vars;

	if (argc ==1) {
		return (1);
	}
	vars.position = malloc(sizeof(int) * count_commands(argv));
	vars.cmds = malloc(sizeof(char **) * count_commands(argv));
	if (!vars.position || !vars.cmds) {
		return (1);
	}
	set_commands(&vars, argv);
	set_stops(&vars);
	exe(&vars, envp);
	free(vars.cmds);
	free(vars.position);
	return (0);
}
```
## GitHub Repo link

For the complete code version of my microshell: [Microshell GitHub repo](url:https://github.com/FrancisL93/microshell_exam_test)