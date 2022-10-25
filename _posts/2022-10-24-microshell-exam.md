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
	int		fd[2]; //2 fds for when we will call pipe(vars->fd)
	int		*position; //If the command is the last or first of a sequence
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
			vars->position[cmd_list - 1] = FIRST; // define FIRST to 1 in header
		if (!strcmp(argv[i - 1], ";"))
			vars->position[cmd_list - 2] = LAST // define LAST to 2 in header
		//First is for when command is the first of a "|" sequence
		//Last is for when command is the last of a "|" sequence
		//When there is only one command it will be reset to LAST by 2nd if
	}
	vars->position[cmd_list - 1] = LAST;
	vars->cmds[cmd_list] = NULL;
}
```
```c
int	ft_strlen(char *str) {
	int count = 0;
	for (int i = 0; str && str[i]; i++) {
		count++;
	}
	return (count);
}

```