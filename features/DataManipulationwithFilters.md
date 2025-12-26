# Using filters to manipulate data
- Filters let you transform JSON data into YAML data, split a URL to extract the hostname, get the SHA1 hash of a string, add or multiply integers, and much more.
- You can use the Ansible-specific filters documented here to manipulate the data, or any of the standard filters shipped with Jinja2.

## Handling undefined variables
- Filters can help you manage missing or undefined variables by providing default or making some variables optional.
- If you configure ansible to ignore most undefined variables, you can mark some variables as requiring values with the `mandatory` filter.

### Providing default values
- Ypu can provide default values for variables directly in your templates using the Jinja2 `default` filter. This is often a better approach than failing if a variable is not defines:
  - `{{ some_variable | default(5) }}`
- Beginning in version 2.8, attempting to access an attribute of an undefined value in Jinja will return another Undefined value, rather than throwing an error immediately.
- This means that you can now simply use a default with a value in a nested data structure `{{ foo.bar.baz | default('DEFAULT') }}` when you do not know if the intermediate values are defined.
- If you want to use default value when variables evaluate to false or an empty string you have to set the second parameter to `true`:
  - `{{ lookup('env', 'MY_USER') | default('admin', true) }}`

### Making variables optional
- By default, Ansible requires values for all variables in a templated expression.
- However, you can make specific module variables optional. For example, you might want to use a system default for some items and control the value for others. 
- To make a module variable optional, set the default value to the special variable `omit`:

    ```yaml
    - name: Touch files with an optional mode
      ansible.builtin.file:
        name: "{{ item.path }}"
        state: touch
        mode: "{{ item.mode | default(omit) }}"
      with_items:
        - path: /tmp/foo
        - path: /tmp/bar
        - path: /tmp/baz
          mode: "0444"
    ```

### Defining mandatory values
- If you configure Ansible to ignore undefined variables, you may want to define some values as mandatory. By default, Ansible fails if a variables in your playbook or command is undefines.
- You can configure Ansible to allow undefines variables by setting `DEFAULT_UNDEFINED_VAR_BEHAVIOUR` to `false`. In this case, you may want to require some variables to be defined. You can dp this with:
  - `{{ variable | mandatory }}`
- The variable value will be used as ism but the template evaluation will raise an error if it is undefined.
- A convenient way of requiring a variable to be overridden is to give it an undefined value using the `undef()` function.

    ```yaml
    galaxy_utl: "https://galaxy.ansible.com"
    galaxy_api_key: "{{ undef(hint='You must specify your Galaxy API key') }}"
    ```
---

## Defining different values for true/false/null (ternary)
- You can create a test, then define one value to use when the test returns true and another when the test returns false
  - `{{ (status == 'needs_restart') | ternary('restart', 'continue') }}`
- In addition, you can define one value to use on true, one value on false and a third value on null
  - `{{ enabled | ternary('no shutdown', 'shutdown', omit) }}`

---

## Managing data types
- You might need to know, change, or set the data type on a variable.
- For example, a registered variable might contain a dictionary when your next task needs a list, or a user prompt might return a string when your playbook needs a boolean value.
- Use the `ansible.builtin.type_debug`, `ansible_builtin.dict2items` and `ansible.builtin.items2dict` filters to manage data types. You can also use the data type itself to cast a value as a specific data type.

### Discovering the data type
- If you are unsure of the underlying Python type of a variable, you can use the `ansible.builtin.type_debug` filter to display it. This is useful in debugging when you need a particular type of variable.
  - `{{ myvar | type_debug }}`
- You should note that, while this may seem like a useful for checking that you have the right type of data in a variable, you should ofthen prefer *type tests*, which will allow you to test for specific data types.

### Transforming strings into lists
- Use `ansible.builtin.split` filter to transform a character/string delimited string into a list of items suitable for looping.
- For example, if you want to split a string variable fruits by commas, you can use:
  - `{{ fruits | split(',') }}`
- String data before applying the filter
  - `fruits: apple,banana,orange`
- List data after applying the filter
  ```yaml
  fruits:
    - apple
    - banana
    - orange
  ```

### Transforming dictionaries into lists
- Use the `ansible.builtin.dict2items` filter to transform a dictionary into a list of items suitable for looping
  - `{{ dict | dict2items }}`
- Dictionary data before applying the filter
    ```yaml
    location:
      city: New York City
      state: New York
      country: USA
    ```
- The List data after applying the filter
    ```yaml
    - key: city
      value: New York City
    - key: state
      value: New York
    - key: country
      value: USA
    ```
- If you want to configure the names of the keys, the `ansible.builtin.dict2items` filters accepts 2 keyword arguments.
- Pass the `key_name` and `value_name` arguments to configure the names of the keys in the list output:
  - `{{ files | dict2items(key_name='file', value_name='path') }}`
- Dictionary data before applying the filter
    ```yaml
    files:
      users: /etc/passwd
      groups: /etc/group
    ```
- List data post applying the fileter
    ```yaml
    - file: users
      path: /etc/passwd
    - file: groups
      path: /etc/group
    ```

### Transforming lists into dictionaries
- Use the `ansible.builtin.items2dict` filter to transform a list into a dictionary, mapping the content into `key:value` pairs:
  - `{{ tags | items2dict }}`
- List data before applying the filter
    ```yaml
    tags:
      - key: application
        value: payment
      - key: environment
        value: dev
    ```

- Dictionary data after applying the filter
    ```yaml
    tags:
      application: payment
      environment: dev
    ```
- Not all lists use `key` to designate keys and `value` to designate values. For example:

    ```yaml
    fruits:
      - fruit: apple
        color: red
      - fruit: pear
        color: yellow
      - fruit: orange
        color: orange
    ```
- In this example, you must pass the `key_name` and `value_name` arguments to configure the transformation. For example:
  - `{{ fruits | items2dict(key_name='fruit', value_name='color') }}`
- If you do not pass these arguments, or do not pass the correct values for your list, you will see `KeyError: key` or `KeyError: my_typo`

### Forcing the data type
- You can cast values a certain types. For example, if you expect the input "True" from a *vars_prompt* and you want Ansible to recognize it as a boolean value instead of a string:

    ```yaml
    - ansible.builtin.debug:
        msg: test
      when: some_string_value | bool
    ```
- If you want to perform a mathematical comparison on a fact, and you want Ansible to recognize it as an integer instead of string:

```yaml
- ansible.builtin.shell: 
    cmd: echo "Only on RedHat 6, derivatives, and later"
  when: ansible_facts['os_family'] == "RedHat" and ansible_facts['lsb']['major_release'] | int >= 6
```
---

## Formatting data: YAML and JSON
- You can switch a data structure in a template from or to JSON or YAML format, with options for formatting, indenting, and loading data.
- The basic filters are occasionally useful for debugging:
  - `{{ some_variable | to_json }}`
  - `{{ some_variable | to_yaml }}`
- For human-readable output, you can use:
  - `{{ some_variable | to_nice_json }}`
  - `{{ some_variable | to_nice_yaml }}`
- You can change the indentation of either format:
  - `{{ some_variable | to_nice_json(indent=2) }}`
  - `{{ some_variable | to_nice_yaml(indent=8) }}`
- The `ansible.builtin.to_yaml` and `ansible.builtin.to_nice_yaml` filters use the PYYAML library which has a default 80 symbol string length limit. That causes an unexpected line break after 90th symbol
- To avoid such behaviour and generate long lines, use the `width` option. You must use a hardcoded number to define the width, instead of a construction like `float("inf")`, because the filter doesn't support proxying Python functions.
  - `{{ some_variable | to_yaml(indent=8, width=1337) }}` 
  - `{{ some_variable | to_nice_yaml(indent=8, width=1337) }}`
- If you are reading in some already formatted data:
  - `{{ some_variable | from_json }}`
  - `{{ some_variable | from_yaml }}`
- For example:
```yaml
tasks:
  - name: Register JSON output as a variable
    ansible.builtin.shell: cat /path/to/file.json
    register: result
  - name: set a variable
    ansible.builtin.set_fact:
      myvar: "{{ result | from_json }}"
```

### Filter to_json and Unicode support
- By default `ansible.builtin.to_json` and `ansible_builtin.to_nice_json` will convert data received to ASCII, so:
  - `{{ 'München' | to_json }}`
- Will return
  - `'M\u00fcnchen'`
- To keep Unicode characters, pass the parameter `ensure_ascii=False` to the filter
  - `{{ 'München' | to_json(ensure_ascii=False) }}`
  - It will return `München`
- To parse multi-document YAML strings, the `ansible.builtin.from_yaml_all` filter is provided. The `ansible.builtin.to_yaml_all` filter will return a generator of parsed YAML documents
- For example

    ```yaml
    ---
    - name: Playbook to test the parsing of multi-document yaml strings
      hosts: localhost
      gather_facts: no
      tasks:
        - name: read the yaml file
          ansible.builtin.set_fact:
            yaml_doc: "{{ lookup('file', '../files/multi_doc_yaml.yml') }}"
        - name: Print the transformed variable
          ansible.builtin.debug:
            msg: "{{ item }}"
          loop: "{{ yaml_doc | from_yaml_all | list }}"
    ```
- When we run the playbook, the result will be like

    ```commandline
    $ ansible-playbook parse_multi_yaml_str.yml
    [WARNING]: No inventory was parsed, only implicit localhost is available
    [WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'
    
    PLAY [Playbook to test the parsing of multi-document yaml strings] *****************************************
    
    TASK [read the yaml file] **********************************************************************************
    ok: [localhost]
    
    TASK [Print the transformed variable] **********************************************************************
    ok: [localhost] => (item={'cities': [{'name': 'Vishakapatnam', 'state': 'Andhra Pradesh'}, {'name': 'Bangalore', 'state': 'Karnataka'}, {'name': 'Hyderabad', 'state': 'Telangana'}]}) => {
        "msg": {
            "cities": [
                {
                    "name": "Vishakapatnam",
                    "state": "Andhra Pradesh"
                },
                {
                    "name": "Bangalore",
                    "state": "Karnataka"
                },
                {
                    "name": "Hyderabad",
                    "state": "Telangana"
                }
            ]
        }
    }
    ok: [localhost] => (item={'places': [{'name': 'Tank Bund', 'state': 'telangana', 'city': 'Hyderabad'}, {'name': 'Mysore Palace', 'state': 'Karnataka', 'city': 'Mysore'}, {'name': 'Tirumala Temple', 'state': 'Andhra Pradesh', 'city': 'Tirupati'}]}) => {                                                                        
        "msg": {
            "places": [
                {
                    "city": "Hyderabad",
                    "name": "Tank Bund",
                    "state": "telangana"
                },
                {
                    "city": "Mysore",
                    "name": "Mysore Palace",
                    "state": "Karnataka"
                },
                {
                    "city": "Tirupati",
                    "name": "Tirumala Temple",
                    "state": "Andhra Pradesh"
                }
            ]
        }
    }
    
    PLAY RECAP *************************************************************************************************
    localhost                  : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
    
    
    ```
---

## Combining and selecting data
- You can combine data from multiple sources and types, and select values from large data structures, giving you precise control over complex data.

### Combining items from multiple lists: zip and zip_longest

- To get a list combining the elements of other lists use `ansible.builtin.zip` 

    ```yaml
    - name: Give me list combo of two lists
      ansible.builtin.debug:
        msg: "{{ [1,2,3,4,5] | zip(['a', 'b', 'c', 'd', 'e']) | list }}"
    
    # => [[1, "a"], [2, "b"], [3, "c"], [4, "d"], [5, "e"]]
    
    - name: Give me the shortest combo of two lists
      ansible.builtin.debug:
        msg: "{{ [1,2,3] | zip(['a', 'b', 'c', 'd', 'e']) | list }}"
    
    # => [[1, "a"], [2, "b"], [3, "c"]]
    ```

- To always exhause all lists use `ansible.builtin.zip_longest`
```yaml
- name: Give me the longest combo of three lists, fill with X
  ansible.builtin.debug:
    msg: "{{ [1,2,3] | zip_longest(['a', 'b', 'c', 'd', 'e'], [21,22,23], fillvalue='X') | list }}"

# => [[1, "a", 21], [2, "b", 22], [3, "c", 23], ["X", "d", "X"], ["X", "e", "X"]]
```

- Similarly to the outpur of the `ansible.builtin.items2dict` filter, these filters can be used to construct a `dict`.
    - `{{ dict( keys_list | zip(values_list)) }}`
- The List data before applying the filter
    ```yaml
    keys_list:
      - apple
      - banana
    values_list:
      - red
      - yellow
    ```
- Dictionary data post applying the filter
    ```yaml
    apple: red
    banana: yellow
    ```

### combining objects and sub elements
- The `ansible.builtin.subelements` filter produces a product of an object and the sub element values of that object, similar to the `ansible.builtin.subelements` lookup.
- This lets you specify individual subelements to use in a template. For example
  - `{{ users | subelements('groups', skip_missing=True) }}`
- Data before applying the filter
```yaml
users:
  - name: alice
    authorized:
      - /tmp/alice/onekey.pub
      - /tmp/alice/twokey.pub
    groups:
      - wheel
      - docker
  - name: bob
    authorized:
      - /tmp/bob/id_rsa.pub
    groups:
      - docker
```
- Data after applying the filter:

```yaml
-
  - name: alice
    authorized:
      - /tmp/alice/onekey.pub
      - /tmp/alice/twokey.pub
    groups:
      - wheel
      - docker
  - wheel
-
  - name: alice
    authorized:
      - /tmp/alice/onekey.pub
      - /tmp/alice/twokey.pub
    groups:
      - wheel
      - docker
  - docker
-
  - name: bob
    authorized:
      - /tmp/bob/id_rsa.pub
    groups:
      - docker
  - docker  
```

- You can use the transformed data with a loop to iterate over the same subelements for multiple objects

```yaml
- name: Set authorized ssh key, extracting just the data from users
  ansible.posix.authorized_key:
    user: "{{ item.0.name }}"
    key: "{{ lookup('file', item.1) }}"
  loop: "{{ users | subelements('authorized') }}"
```

### Combining hashes / dictionaries
- The `ansible.builtin.combine` filter allows hashes to be merged. For example, the following would override keys in one hash:
  - `{{ {'a': 1, 'b': 2} | combine({'b': 3}) }}`
  - Result will be `{ 'a': 1, 'b': 3 }`
- The filter can also take multiple arguments to merge
- Example

    ```yaml
    ---
    - name: ansible combine filter
      hosts: localhost
      gather_facts: false
      vars:
        a:
          1: 10
          2: 20
          3: 30
        b:
          3: 31
          4: 41
        c:
          4: 42
          5: 52
        d:
          5: 53
      tasks:
        - name: combine multiple elements
          ansible.builtin.debug:
            msg: "{{ a | combine(b, c, d) }}"
        - name: combine a list of items
          ansible.builtin.debug:
            msg: "{{ [a, b, c, d] | combine }}"
    ```
    ```commandline
    $ ansible-playbook combine_filter.yml
    [WARNING]: No inventory was parsed, only implicit localhost is available
    [WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'
    
    PLAY [ansible combine filter] ******************************************************************************
    
    TASK [combine multiple elements] ***************************************************************************
    ok: [localhost] => {
        "msg": {
            "1": 10,
            "2": 20,
            "3": 31,
            "4": 42,
            "5": 53
        }
    }
    
    TASK [combine a list of items] *****************************************************************************
    ok: [localhost] => {
        "msg": {
            "1": 10,
            "2": 20,
            "3": 31,
            "4": 42,
            "5": 53
        }
    }
    
    PLAY RECAP *************************************************************************************************
    localhost                  : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
    
    ```
- The filter also accepts two optional parameters: `recursive` and `list_merge`
- **recursive**
  - Is a boolean, default to `False`. Should the `ansible.builtin.combine` recursively merge nested hashes. Note: It does not depend on the value of the `hash_behaviour` settings in `ansible.cfg`
- **list_merge**
  - Is a string, its possible values are `replace` (default), `keep`, `append`, `prepend`, `append_rp` or `prepend_rp`. It modifies the behaviour of the filter when the hashes to merge contain arrays/lists.

- Consider the below example:

    ```yaml
    default:
      a:
        x: default
        y: default
      b: default
      c: default
    patch:
      a:
        y: patch
        z: patch
      b: patch
    ```
- If `recursive=False` (the default), nested hash aren't merged:
  - `{{ default | combine(patch) }}`
- The result will be
    ```yaml
    a:
      y: patch
      z: patch
    b: patch
    c: default
    ```
- If `recursive=True`, recurse into a nested hash and merge their keys
  - `{{ default | combine(patch, recursive=True) }}`
- The result will be

    ```yaml
    a: 
      x: default
      y: patch
      z: patch
    b: patch
    c: default
    ```
- If `list_merge='replace'` (the default), arrays from the right hash will "replace" the ones in the left hash:
    
    ```yaml
    default:
      a:
        - default
    patch:
      a:
        - patch
    ```
  - `{{ default | combine(patch) }}` 
- This will result in
    ```yaml
    a:
      - patch
    ```
  
- If `list_merge='keep'`, arrays from the left hash will be kept:
  - `{{ default | combine(patch, list_merge='keep') }}`
- This would result in
    ```yaml
    a:
      - default
    ```

- If `list_merge='append'`, arrays from right hash will be appended to the ones in the left hash:
  - `{{ default | combine(patch, list_merge='append') }}`
- This would result in
    ```yaml
    a: 
      - default
      - patch
    ```

- If `list_merge='prepend'`, arrays from the right hash will be prepended to the ones in the left hash
  - `{{ default | combine(patch, list_merge='prepend') }}`
- This would result in
    ```yaml
    a:
      - patch
      - default
    ```

- If `list_merge='append_rp'`, arrays from the right hash will be appended to the ones in the left hash. Elements of arrays in the left hash that are also in the corresponding array of the right hash will be removed. Duplicate elements that aren't in both hashes are kept:

    ```yaml
    default:
      a:
        - 1
        - 1
        - 2
        - 3
    patch:
      a:
        - 3
        - 4
        - 5
        - 5  
    ```
  - `{{ default | combine(patch, list_merge='append_rp') }}`
- This would result in

    ```yaml
    a:
      - 1
      - 1
      - 2
      - 3
      - 4
      - 5
      - 5 
    ```
- If `list_merge='prepend_rp'`, the behaviour is similar  to the one for `append_rp`. but elements of arrays in the right hash are prepended:
  - `{{ default | combine(patch, list_merge='prepend_rp') }}`
- This would result in
    ```yaml
    a:
      - 3
      - 4
      - 5
      - 5
      - 1
      - 1
      - 2 
    ```
- We can also use `recursive` and `list_merge` together
    
    ```yaml
    default:
      a:
        a':
          x: default_value
          y: default_value
          list:
            - default_value
      b:
        - 1
        - 1
        - 2
        - 3
    patch:
      a:
        a':
          y: patch_value
          z: patch_value
          list:
            - patch_value
      b:
        - 3
        - 4
        - 4
        - key: value
    ```
  - `{{ default | combine(patch, recursive=True, list_merge='append_rp') }}`
- This would result in

    ```yaml
    a:
      a':
        x: default_value
        y: patch_value
        z: patch_value
        list:
          - default_value
          - patch_value
      b:
        - 1
        - 1
        - 2
        - 3
        - 4
        - 4
        - key: value
    ```

### Selecting values form arrays or hashtables

- The *extract* filter used to map from a list of indices to a list values from a container (hash or array)

    ```commandline
    {{ [0,2] | map('extract', ['x', 'y', 'z']) | list ) }}
    {{ ['x', 'y'] | map('extract', {'x': 42, 'y': 31}) | list }}
    ```
- The results of above expressions would be:
    ```commandline
    ['x', 'z']
    [42, 31]
    ```
- The filter can take another argument:
  - `{{ groups['x'] | map('extract', hostvars, 'ec2_ip_address') | list }}`
- This takes the list of hosts in group 'x', looks them up in *hostvars*, and then looks up the *ec2_ip_address* of the result. The final result is a list of IP addresses for the hosts in group 'x'.
- The third argument to the filter can also be a list, for recursive lookup inside the container:
  - `{{ ['a'] | map('extract', b, ['x', 'y']) | list }}`
- This would result a list containing rhe value of *b['a']['x']['y']*

### Combining lists
- This set of filters returns a list of combines lists

#### Permutations
- To get permutations of a list:

    ```yaml
    - name: Ansible List permutations
      hosts: localhost
      gather_facts: false
      tasks:
        - name: Generating the largest permutations
          ansible.builtin.debug:
            msg: "{{ [1,2,3,4,5] | ansible.builtin.permutations | to_json }}"
    
        - name: Generating permutation of sets of three
          ansible.builtin.debug:
            msg: "{{ [1,2,3,4,5] | ansible.builtin.permutations(3) | to_json }}"
    
    ```
    
    ```commandline
    $ ansible-playbook permutations.yml
    [WARNING]: No inventory was parsed, only implicit localhost is available
    [WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'
    
    PLAY [Ansible List permutations] ***************************************************************************
    
    TASK [Generating the largest permutations] *****************************************************************
    ok: [localhost] => {
        "msg": "[[1, 2, 3, 4, 5], [1, 2, 3, 5, 4], [1, 2, 4, 3, 5], [1, 2, 4, 5, 3], [1, 2, 5, 3, 4], [1, 2, 5, 4, 3], [1, 3, 2, 4, 5], [1, 3, 2, 5, 4], [1, 3, 4, 2, 5], [1, 3, 4, 5, 2], [1, 3, 5, 2, 4], [1, 3, 5, 4, 2], [1, 4, 2, 3, 5], [1, 4, 2, 5, 3], [1, 4, 3, 2, 5], [1, 4, 3, 5, 2], [1, 4, 5, 2, 3], [1, 4, 5, 3, 2], [1, 5, 2, 3, 4], [1, 5, 2, 4, 3], [1, 5, 3, 2, 4], [1, 5, 3, 4, 2], [1, 5, 4, 2, 3], [1, 5, 4, 3, 2], [2, 1, 3, 4, 5], [2, 1, 3, 5, 4], [2, 1, 4, 3, 5], [2, 1, 4, 5, 3], [2, 1, 5, 3, 4], [2, 1, 5, 4, 3], [2, 3, 1, 4, 5], [2, 3, 1, 5, 4], [2, 3, 4, 1, 5], [2, 3, 4, 5, 1], [2, 3, 5, 1, 4], [2, 3, 5, 4, 1], [2, 4, 1, 3, 5], [2, 4, 1, 5, 3], [2, 4, 3, 1, 5], [2, 4, 3, 5, 1], [2, 4, 5, 1, 3], [2, 4, 5, 3, 1], [2, 5, 1, 3, 4], [2, 5, 1, 4, 3], [2, 5, 3, 1, 4], [2, 5, 3, 4, 1], [2, 5, 4, 1, 3], [2, 5, 4, 3, 1], [3, 1, 2, 4, 5], [3, 1, 2, 5, 4], [3, 1, 4, 2, 5], [3, 1, 4, 5, 2], [3, 1, 5, 2, 4], [3, 1, 5, 4, 2], [3, 2, 1, 4, 5], [3, 2, 1, 5, 4], [3, 2, 4, 1, 5], [3, 2, 4, 5, 1], [3, 2, 5, 1, 4], [3, 2, 5, 4, 1], [3, 4, 1, 2, 5], [3, 4, 1, 5, 2], [3, 4, 2, 1, 5], [3, 4, 2, 5, 1], [3, 4, 5, 1, 2], [3, 4, 5, 2, 1], [3, 5, 1, 2, 4], [3, 5, 1, 4, 2], [3, 5, 2, 1, 4], [3, 5, 2, 4, 1], [3, 5, 4, 1, 2], [3, 5, 4, 2, 1], [4, 1, 2, 3, 5], [4, 1, 2, 5, 3], [4, 1, 3, 2, 5], [4, 1, 3, 5, 2], [4, 1, 5, 2, 3], [4, 1, 5, 3, 2], [4, 2, 1, 3, 5], [4, 2, 1, 5, 3], [4, 2, 3, 1, 5], [4, 2, 3, 5, 1], [4, 2, 5, 1, 3], [4, 2, 5, 3, 1], [4, 3, 1, 2, 5], [4, 3, 1, 5, 2], [4, 3, 2, 1, 5], [4, 3, 2, 5, 1], [4, 3, 5, 1, 2], [4, 3, 5, 2, 1], [4, 5, 1, 2, 3], [4, 5, 1, 3, 2], [4, 5, 2, 1, 3], [4, 5, 2, 3, 1], [4, 5, 3, 1, 2], [4, 5, 3, 2, 1], [5, 1, 2, 3, 4], [5, 1, 2, 4, 3], [5, 1, 3, 2, 4], [5, 1, 3, 4, 2], [5, 1, 4, 2, 3], [5, 1, 4, 3, 2], [5, 2, 1, 3, 4], [5, 2, 1, 4, 3], [5, 2, 3, 1, 4], [5, 2, 3, 4, 1], [5, 2, 4, 1, 3], [5, 2, 4, 3, 1], [5, 3, 1, 2, 4], [5, 3, 1, 4, 2], [5, 3, 2, 1, 4], [5, 3, 2, 4, 1], [5, 3, 4, 1, 2], [5, 3, 4, 2, 1], [5, 4, 1, 2, 3], [5, 4, 1, 3, 2], [5, 4, 2, 1, 3], [5, 4, 2, 3, 1], [5, 4, 3, 1, 2], [5, 4, 3, 2, 1]]"                                                                                                           
    }
    
    TASK [Generating permutation of sets of three] *************************************************************
    ok: [localhost] => {
        "msg": "[[1, 2, 3], [1, 2, 4], [1, 2, 5], [1, 3, 2], [1, 3, 4], [1, 3, 5], [1, 4, 2], [1, 4, 3], [1, 4, 5], [1, 5, 2], [1, 5, 3], [1, 5, 4], [2, 1, 3], [2, 1, 4], [2, 1, 5], [2, 3, 1], [2, 3, 4], [2, 3, 5], [2, 4, 1], [2, 4, 3], [2, 4, 5], [2, 5, 1], [2, 5, 3], [2, 5, 4], [3, 1, 2], [3, 1, 4], [3, 1, 5], [3, 2, 1], [3, 2, 4], [3, 2, 5], [3, 4, 1], [3, 4, 2], [3, 4, 5], [3, 5, 1], [3, 5, 2], [3, 5, 4], [4, 1, 2], [4, 1, 3], [4, 1, 5], [4, 2, 1], [4, 2, 3], [4, 2, 5], [4, 3, 1], [4, 3, 2], [4, 3, 5], [4, 5, 1], [4, 5, 2], [4, 5, 3], [5, 1, 2], [5, 1, 3], [5, 1, 4], [5, 2, 1], [5, 2, 3], [5, 2, 4], [5, 3, 1], [5, 3, 2], [5, 3, 4], [5, 4, 1], [5, 4, 2], [5, 4, 3]]"                                                                                   
    }
    
    PLAY RECAP *************************************************************************************************
    localhost                  : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
    
    ```

#### Combinations
- Combinations always require a set size

    ```yaml
    - name: Generate the combinations for set size of two
      ansible.builtin.debug:
        msg: "{{ [1,2,3,4,5] | ansible.builtin.combinations(2) | list }}"
    
    # => [(1, 2), (1, 3), (1, 4), (1, 5), (2, 3), (2, 4), (2, 5), (3, 4), (3, 5), (4, 5)]
    ```
#### Products
- THe product filter returns the cartesian product of the input iterables. This is roughly equivalent to nested for-loops in a generator expression.
    
    ```
    {{ ['foo', 'bar'] | product(['com']) }}
    # => [('foo', 'com'), ('bar', 'com')] 
    ```
- For example to generate multiple hostnames
    ```
    {{ ['foo', 'bar'] | product(['com']) | map('join', '.') }}
    
    # => ['foo.com', 'bar.com']
    ```

### Selecting JSON data: JSON queries
- To select a single element or a data subset from a complex data structure in JSON format (for example, Ansible facts), use the `community.general.json_query` filter.
- This filter lets you query a complex JSON structure and iterate over it using a loop structure.
- Consider the below example

    ```yaml
    ---
    - name: Extract data using json_query module
      hosts: localhost
      gather_facts: false
      vars:
        qry_port: "domain.server[?cluster=='cluster1'].port"
      tasks:
        - name: load the json data
          ansible.builtin.set_fact:
            domain_data: "{{ lookup('file', '../files/domain_data.json') | from_json }}"
    
        - name: Log the json data
          ansible.builtin.debug:
            msg: "{{ domain_data }}"
    
        - name: Extract and display all the cluster names
          ansible.builtin.debug:
            msg: "{{ domain_data | community.general.json_query('domain.cluster[*].name') | list }}"
    
        - name: Extract and display all the server names
          ansible.builtin.debug:
            msg: "{{ domain_data | community.general.json_query('domain.server[*].name') | list }}"
    
        - name: Extract the ports from cluster1
          ansible.builtin.debug:
            msg: "{{ domain_data | community.general.json_query(qry_port) | list }}"
    
    
        - name: Display all the ports with comma separated values
          ansible.builtin.debug:
            msg: "{{ domain_data | community.general.json_query(qry_port)  | join(',') }}"
    
        - name: Display all the server ports and names from cluster1
          ansible.builtin.debug:
            msg: "{{ domain_data | community.general.json_query('domain.server[?cluster==`cluster1`].{name: name, port: port}') }}"
    
        - name: Display ports from all clusters with name starting with 'server1'
          ansible.builtin.debug:
            msg: "{{ domain_data | community.general.json_query('domain.server[?starts_with(name, `server1`)].port') }}"
    ```
    
    ```commandline
    $ ansible-playbook extract_using_json_query.yml
    [WARNING]: No inventory was parsed, only implicit localhost is available
    [WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'
    
    PLAY [Extract data using json_query module] ****************************************************************
    
    TASK [load the json data] **********************************************************************************
    ok: [localhost]
    
    TASK [Log the json data] ***********************************************************************************
    ok: [localhost] => {
        "msg": {
            "domain": {
                "cluster": [
                    {
                        "name": "cluster1"
                    },
                    {
                        "name": "cluster2"
                    }
                ],
                "library": [
                    {
                        "name": "lib1",
                        "target": "cluster1"
                    },
                    {
                        "name": "lib2",
                        "target": "cluster2"
                    }
                ],
                "server": [
                    {
                        "cluster": "cluster1",
                        "name": "server11",
                        "port": "8080"
                    },
                    {
                        "cluster": "cluster1",
                        "name": "server12",
                        "port": "8090"
                    },
                    {
                        "cluster": "cluster2",
                        "name": "server21",
                        "port": "9080"
                    },
                    {
                        "cluster": "cluster2",
                        "name": "server22",
                        "port": "9090"
                    }
                ]
            }
        }
    }
    
    TASK [Extract and display all the cluster names] ***********************************************************
    ok: [localhost] => {
        "msg": [
            "cluster1",
            "cluster2"
        ]
    }
    
    TASK [Extract and display all the server names] ************************************************************
    ok: [localhost] => {
        "msg": [
            "server11",
            "server12",
            "server21",
            "server22"
        ]
    }
    
    TASK [Extract the ports from cluster1] *********************************************************************
    ok: [localhost] => {
        "msg": [
            "8080",
            "8090"
        ]
    }
    
    TASK [Display all the ports with comma separated values] ***************************************************
    ok: [localhost] => {
        "msg": "8080,8090"
    }
    
    TASK [Display all the server ports and names from cluster1] ************************************************
    ok: [localhost] => {
        "msg": [
            {
                "name": "server11",
                "port": "8080"
            },
            {
                "name": "server12",
                "port": "8090"
            }
        ]
    }
    
    TASK [Display ports from all clusters with name starting with 'server1'] ***********************************
    ok: [localhost] => {
        "msg": [
            "8080",
            "8090"
        ]
    }
    
    PLAY RECAP *************************************************************************************************
    localhost                  : ok=8    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
    
    ```

## Hashing and encrypting strings and passwords

- To get the sha1 hash of a string
    ```
    {{ 'test1' | hash('sha1') }}`
    # => "b444ac06613fc8d63795be9ad0beaf55011936ac"
    ```
- To get the md5 hash of a string
    ```
    {{ 'test1' | hash('md5') }}
    # => "5a105e8b9d40e1329780d62ea2265d8a"
    ```
- Get a string checksum:
    ```
    {{ 'test1' | checksum }}
    # => "109f4b3c50d7b0df729d299bc6f8e9ef9066971f"
    ```
- Other hashes (platform dependent)
    ```
    {{ 'test2' | hash('blowfish') }}
    ```
- To get a sha512 password hash (random salt)
    ```
    {{ 'passwordsaresecret' | password_hash('sha512') }}
    # => "$6$UIv3676O/ilZzWEE$ktEfFF19NQPF2zyxqxGkAceTnbEgpEKuGBtk6MlU4v2ZorWaVQUMyurgmHCh2Fr4wpmQ/Y.AlXMJkRnIS4RfH/"
    ```
- To get a sha256 password hash with a specific salt

    ```
    {{ 'secretpassword' | password_hash('sha256', 'mysecretsalt') }}
    # => "$5$mysecretsalt$ReKNyDYjkKNqRVwouShhsEqZ3VOE8eoVO4exihOfvG4"
    ```
- An idempotent method to generate unique hashes per system is to use a salt that is consistent between runs:
```
{{ 'secretpassword' | password_hash('sha512', 65534 | random(seed=inventory_hostname) | string }}
# => "$6$43927$lQxPKz2M2X.NWO.gK.t7phLwOKQMcSq72XxDZQ0XzYV6DlL1OD72h417aj16OnHTGxNzhftXJQBcjbunLEepM0"
```

- Hash types available depend on the control system running Ansible, `ansible.builtin.hash` depends on hashlib, `ansible.builtin.password_hash` depends on `passlib`. The `crypt` is used as a fallback if `passlib` is not installed.
- Some hash types allow providing a rounds parameter"
  - `{{ 'secretpassword' | password_hash('sha256', 'mysecretsalt', rounds=10000) }}`
- The filter *password_hash* produces different results depending on whether you installed *passlib* or not.
- To ensure idempotency, specify rounds to be neigher *crypt's* not *passlib's* default, which is 5000 for *crypt* and a variable value (535000 for sha256, 656000 for sha512) for *passlib*.

- Hash type 'blowfish' (BCrypt) provides the facility to specify the version of the BCrypt algorithm.
  - `{{ 'secretpassword' | password_hash('blowfish', '12345456', ident='2b') }}`
- NOTE: The parameter is only available for blowfish (BCrypt). Other hash types will simply ignore this parameter. Valid values for this parameter are : `['2', '2a', '2y', '2b']`

- You can also use the Ansible `ansible.builtin.vault` filter to encrypt and `ansible.builtin.unvault` fileter to decrypt.

```yaml
vars:
  myvaultedkey: "{{ keyrawdata | vault(passphrase) }}"

tasks:
  - name: Saved templated vault data
    ansible.builtin.template:
      src: dump_template_file.j2
      dest: /some/key/vault.txt
    vars:
      mysalt: "{{ 2**256 | random(seed=inventory_hostname) }}"
      template_data: "{{ secretdata | vault(vaultsecret, salt=mysalt) }}"
```

```yaml
vars:
  mykey: "{{ myvaultedkey | unvault(passphrase) }}"
tasks:
  - name: saved templated unvaulted data
    ansible.builtin.template:
      src: dump_template_data.j2
      dest: /some/key/clear.txt
    vars:
      template_data: "{{ secretdata | unvault(vaultsecret) }}"
```

---

## Manipulating text
- Several filters work with text, including URLs, filenames, and path names.

### Adding comments to files
- The ansible.builtin.comment filter lets you create comments in a file from text in a template, which a variety of comment styles.
- By default, Ansible uses `#` to start the comment line and adds a blank comment line above and below your comment text. For example
  - `{{ "Plain style (default)" | comment }}`
- produces this output
  ```
  #
  # Plain style (default)
  #
  ```
- Ansible offers styles for comments in C (`//...`), C block (`/*...*/`), Erlang (`%...`) and XML (`<!-- ... -->`)
    ```
    {{ "C style" | comment('c') }}
    {{ "C block style" | comment('cblock') }}
    {{ "Erlang style" | comment('erlang') }}
    {{ "XML Style" | comment('xml') }}
    ```
- You can define a custom comment character. This filter:
  - `{{ "My special case" | comment(decoration="! ") }}`
- Produces
    ```
    !
    ! My special case
    !
    ```
- You can fully customize the comment style:
  - `{{ "Custom Style" | comment('plain', prefix='#####\n', postfix='\n#####') }}`
- This will create the following type of output
    ```
    #####
    # Custom Style
    #####
    ```
### URLEncode Variables
- The `urlencode` filter quotes data for use ina URL path or query using UTF-8:
```
{{ 'Trollhättan' | urlencode }}
# =>  'Trollh%C3%A4ttan'
```

### Splitting URLs
- The `ansible.builtin.urlsplit` filter extracts the fragment, hostname, netloc, password, port, query, scheme, and username from a URL with no arguments, returns a dictionary of all the fileds:

```
vars:
  my_url: "http://user:password@www.acme.com:9000/dir/index.html?query=term#fragment"

{{ my_url | urlsplit('hostname') }}
#=> 'www.acme.com'

{{ my_url | urlsplit('netloc') }}
#=> 'user:password@www.acme.com:9000'

{{ my_url | urlsplit('username') }}
#=> 'user'

{{ my_url | urlsplit('password') }}
#=> 'password'

{{ my_url | urlsplit('path') }}
#=> '/dir/index.html'

{{ my_url | urlsplit('port') }}
#=> 9000

{{ my_url | urlsplit('scheme') }}
#=> 'http'

{{ my_url | urlsplit('query') }}
#=> 'query=term'

{{ my_url | urlsplit('fragment') }}
#=> 'fragment'

{{ my_url | urlsplit }}

# =>
#   {
#       "fragment": "fragment",
#       "hostname": "www.acme.com",
#       "netloc": "user:password@www.acme.com:9000",
#       "password": "password",
#       "path": "/dir/index.html",
#       "port": 9000,
#       "query": "query=term",
#       "scheme": "http",
#       "username": "user"
#   }

```

### Searching strings with regular expressions
- To search in a string or extract parts of a string with a regular expression, use the `ansible.builtin.regex_search` filter:

    ```
    # Extracts the database name from a string
    {{ 'server1/database42' | regex_search('database[0-9]+') }}
    #=> 'database42'
    
    # Example for a case insensitive search in multiline mode
    {{ 'foo/nBAR' | regex_search('^bar', multiline=True, ignorecase=True) }}
    #=> 'BAR'
    
    # Example for a case insensitive search in multiline mode using inline regex flags
    {{ 'foo\nBAR' | regex_search('(?im)^bar') }}
    #=> 'BAR'
    
    # extracts server and database id from a string
    {{ 'server1/database42' | regex_search('server([0-9]+)/database([0-9]+)'), '\\1', '\\2' }}
    #=> ['1', '42']
    
    # Extract dividind and a divisor from a division
    {{ '21/42' | regex_search('(?P<dividend>[0-9]+)/(?P<divisor>[0-9]+)', '\\d<dividend>', '\\g<divisor>') }}
    #=> ['21', '42']
    ```
- To find all occurrences of regex matches in a string, use the `ansible.builin.regex_findall` filter:

    ```
    # returns a list of all IPV4 addresses in the string
    
    {{ 'Some DNS servers are 8.8.8.8 and 169.254.0.1' | regex_findall('\\b(?:[0-9]{1,3}\\.){3}[0-9]{1,3}\\b') }}
    #=> ['8.8.8.8', '169.254.0.1']
    
    # return all lines that end with "ar"
    {{ 'CAR\ntar\nfoo\nbar\n' | regex_findall('^.ar$', multiline=True, ignorecase=True) }}
    #=> ['CAR', 'tar', 'bar']
    
    # Returns all lines that end with "ar" using inline regex flags for multiline and ignorecase
    {{ 'CAR\ntar\nfoo\nbar\n' | regex_findall('(?im)^.ar$') }}
    #=> ['CAR', 'tar', 'bar']
    ```

- To replace text in a string with regex, use the `ansible.builtin.regex_replace` filter:

    ```
    # Converts "ansible" to "able"
    {{ 'ansible' | regex_replace('^a.*i(.*)$', 'a\\1') }}
    #=> 'able'
    
    # converts "foobar" to "bar"
    {{ 'foobar' | regex_replace('^f.*o(.*)$', '//1') }}
    # => 'bar'
    
    Converts "http://localhost:80" to 'http, localhost, 80'
    {{ 'http://localhost:80' | regex_replace( '^(?P<protocol>\\w+)://(?P<host>\\w+):(?P<port>\\d+)$', '\\g<protocol>, \\g<host>, \\g<port>' ) }}
    #=> "http, localhost, 80"
    
    # Comment all lines that end with "ar"
    {{ 'CAR\ntar\nfoo\nbar\n' | regex_replace('(?im)^(.*ar)$', '#\\1', multiline=True, ignorecase=True) }}
    # => "#CAR\n#tar\nfoo\n#bar\n"
    
    
    # Comment all lines that end with "ar" using inline filters
    {{ 'CAR\ntar\nfoo\nbar\n' | regex_replace('(?im)^(.*ar)$', '#\\1') }}
    # => "#CAR\n#tar\nfoo\n#bar\n"
    
    ```
  
- To escape special characters within a standard Python regex, use the `ansible.builtin.regex_escape` filter (using the default `re_type='python'` option):
    ```
    # convert '^f.*o(.*)$' to '\^f\.\*o\(\.\*\)\$'
    {{ '^f.*o(.*)$' | regex_escape() }}
    #=> "\\^f\\.\\*o\\(\\.\\*\\)\\$"
    ```

- To escape special characters within a POSIX basic regex, use the `ansible.builtin.regex_escape` filter with the `re_type='posix_basic'` option:
    ```
    {{ '^f.*o(.*)$' | regex_escape(re_type='posix_basic') }}
    #=> "\\^f\\.\\*o(\\.\\*)\\$"
    ```

### Managing file names and path names
- To get the last name of a file path, like *foo.txt* out ot */etc/asdf/foo.txt*
  - `{{ path | basename }}`
- To get the last name of a windows style file path
  - `{{ path | win_basename }}`
- To separate the windows drive letter from the rest of a file path 
  - `{{ path | win_splitdrive }}`
- To get only the windows drive letter
  - `{{ path | win_splitdrive | first }}`
- To get the rest of the path without the drive letter
  - `{{ path | win_splitdrive | last }}`
- To get the directory from a path
  - `{{ path | dirname }}`
- To get the directory from a windows path
  - `{{ path | win_dirname }}`
- To expand a path containing a tilde (~) character
  - `{{ path | expanduser }}`
- To expand a path containing environment variable (expandvars expands local variables; using it on remote paths can lead to errors)
  - `{{ path | expandvars }}`
- To get the real path of a link
  - `{{ path | realpath }}`
- To get the relative path of a link, from a start point
  - `{{ path | relpath('/etc') }}`
- To get the root and extension of a path or file name
    ```
    # with path = nginx.conf the reutrn would be ('nginx', '.conf')
    {{ path | splitext }}
    ```
- The ansible.builtin.splitext filter always returns a pair of strings. The individual components can be accessed by using the `first` and `last` filter.

    ```
    # with path=nginx.conf
    {{ path | splitext | first }} # => nginx
    {{ path | splitext | last }} # => .conf
    ```

---

## Manipulating strings
- To add quotes for shell usage

    ```yaml
    - name: Run a shell command
      ansible.builtin.shell: echo {{ string_value | quote }}
    ```
- To concatenate a list into a string
  - `{{ list | join(" ") }}`
- To split a string into a list
  - `{{ csv_str | split(',') }}`
- To work with base64 encoded strings:

```
{{ encoded | b64decode }}
{{ decoded | string | b64encode }}
```