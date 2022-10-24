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
**char \*\*envp:** environement variables stocked as envp[...]
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
	//In exam, I don't care for malloc errors...
}
```
### 5-

```c
int	ft_strlen(char *str) {
	int count = 0;
	for (int i = 0; str && str[i]; i++) {
		count++;
	}
	return (count);
}

```