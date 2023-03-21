#!/bin/bash

declare -A nodes childNodes nodeTypes identifier values
declare -a focus
focus=("")
shopt -s extglob

refocus() {
  local -i level difference depth
  local -a nodeArgs
  local node nodeType address parent
  level=$1
  nodeId="$2"
  file="$3"
  nodeType="$4"
  nodeArgs=("${@:4}")
  depth=${#focus[@]}
  difference=$((depth-level))
  
  while [ $difference -gt 2 ]
  do
    unset 'focus[-1]'
    depth=${#focus[@]}
    difference=$((depth-level))
  done
  
  if [ $difference = 2 ]
  then
    unset 'focus[-1]'
    difference=$((difference-1))
  fi
  if [ $difference = 1 ]
  then
    focus+=("$nodeId")
  fi
  
  printf -v address ".%s" "${focus[@]}"
  parent="${address%.*}"
  address="${address:2}"
  parent="${parent:2}"
  nodeTypes["$file:$address"]="${nodeType}"
  
  if [ "${values["$file:$address"]}" = "" ]
  then values["$file:$address"]="${nodeArgs[*]:1}"
  else values["$file:$address"]+=" ${nodeArgs[*]:1}"
  fi

  if [ "${childNodes["$file:$parent"]}" = "" ]
  then childNodes["$file:$parent"]="$file:$address"
  else childNodes["$file:$parent"]+=" $file:$address"
  fi
}

parseLine() {
  local -i level abort lineNo
  local -a atoms
  local line IFS id file
  file="$1"
  line="$2"
  level=0
  abort=0

  while [ $abort = 0 ]
  do
    case "$line" in
      *( )#*|*( ))
        abort=1
        lineNo=$((lineNo+1))
        ;;
      '  '*)
        level=$((level+1))
        line="${line:2}"
        lineNo=$((lineNo+1))
        ;;
      *)
        read -ra atoms <<< "$line"
        nonEmpty "${identifier[${atoms[0]}]}" "Unrecognized identifier: ${atoms[0]} in $file line $lineNo"
        id="${atoms[${identifier[${atoms[0]}]}]}"
        nonEmpty "$id" "Unrecognized atom: '${atoms[0]}'"
        refocus $level "$id" "$file" "${atoms[0]}" "${atoms[@]:1}"
        abort=1
        lineNo=$((lineNo+1))
        ;;
    esac
  done
}

parseCodl() {
  local file
  file="$1"
  while IFS=$'\n' read -r line
  do parseLine "$file" "$line"
  done < "$file"
}

childPaths() {
  local filter node IFS
  local -a nodes
  IFS=$' '
  node="$1"
  filter="$2"
  read -ra nodes <<< "${childNodes[$node]}"
  for node in "${nodes[@]}"
  do
    if [ "$filter" = "" ]
    then printf "%s\n" "$node"
    elif [ "$filter" = "${nodeTypes[$node]}" ]
    then printf "%s\n" "$node"
    fi
  done
}

children() {
  local filter node IFS
  local -a nodes
  local -i prefix
  IFS=$' '
  node="$1"
  prefix="${#node}"
  filter="$2"
  
  if [ "${node:0-1}" != ":" ]
  then prefix=$((prefix+1))
  fi
  
  read -ra nodes <<< "${childNodes[$node]}"
  for node in "${nodes[@]}"
  do
    if [ "$filter" = "" ]
    then printf "%s\n" "${node:$prefix}"
    elif [ "$filter" = "${nodeTypes[$node]}" ]
    then printf "%s\n" "${node:$prefix}"
    fi
  done
}

nonEmpty() {
  if [ "$1" = "" ]
  then
    printf "%s\n" "$2"
    exit 1
  fi
}

values() {
  local address
  local -a data
  address="$1"
  nonEmpty "$address" "Cannot access values for empty node"
  read -ra data <<< "${values[$address]}"
  for datum in "${data[@]}"
  do printf "%s\n" "$datum"
  done
}