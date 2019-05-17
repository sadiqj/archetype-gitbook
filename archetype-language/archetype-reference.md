# Reference

{% hint style="warning" %}
coming soon
{% endhint %}

## Grammar


 
    <style>
      .specification td, th{
          vertical-align: baseline;
          padding: 0;
          margin: 0;
          font-weight: normal;
      }
      .specification td {
          text-align: left;
      }
      .specification th {
          text-align: right;
          white-space: nowrap;
      }
      .specification th::after {
          content: "\a0::=\a0";
      }
      .specification th.bar {
          text-align: right;
      }
      .specification th.bar::after {
          content: "|\a0";
      }
      .rule th, td {
          padding-top: .5em;
      }
      .nonterminal::before {
          content: "<";
      }
      .nonterminal::after {
          content: ">";
      }
      .list::after {
          content: "*";
          vertical-align: super;
          font-size: smaller;
      }
      .ne_list::after {
          content: "+";
          vertical-align: super;
          font-size: smaller;
      }
      .option::before {
          content: "[";
      }
      .option::after {
          content: "]";
      }
    </style>
  </head>
  
  <body>
    
    <table class="specification">
      
      <tr class="rule">
        <th><span class="nonterminal">loc(X)</span></th>
        <td>X</td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">snl(separator, X)</span></th>
        <td>X</td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td>X
        separator</td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td>X
        separator
        <span class="nonterminal">snl(separator,
        X)</span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">snl2(separator, X)</span></th>
        <td>X
        separator
        X</td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td>X
        separator
        <span class="nonterminal">snl2(separator,
        X)</span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">paren(X)</span></th>
        <td>LPAREN
        X
        RPAREN</td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">braced(X)</span></th>
        <td>LBRACE
        X
        RBRACE</td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">bracket(X)</span></th>
        <td>LBRACKET
        X
        RBRACKET</td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">start_expr</span></th>
        <td><span class="nonterminal">expr</span>
        EOF</td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td><span class="nonterminal">loc(error)</span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">main</span></th>
        <td><span class="nonterminal">loc(<span class="nonterminal">archetype_r</span>)</span></td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td><span class="nonterminal">loc(error)</span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">archetype_r</span></th>
        <td><span class="nonterminal">implementation_archetype</span>
        EOF</td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td><span class="nonterminal">archetype_extension</span>
        EOF</td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">archetype_extension</span></th>
        <td>ARCHETYPE
        EXTENSION
        <span class="nonterminal">ident</span>
        <span class="nonterminal">paren(<span class="nonterminal">declarations</span>)</span>
        EQUAL
        <span class="nonterminal">braced(<span class="nonterminal">declarations</span>)</span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">implementation_archetype</span></th>
        <td><span class="nonterminal">declarations</span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">declarations</span></th>
        <td><span class="ne_list"><span class="nonterminal">declaration</span></span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">declaration</span></th>
        <td><span class="nonterminal">loc(<span class="nonterminal">declaration_r</span>)</span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">declaration_r</span></th>
        <td><span class="nonterminal">archetype</span></td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td><span class="nonterminal">constant</span></td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td><span class="nonterminal">variable</span></td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td><span class="nonterminal">enum</span></td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td><span class="nonterminal">states</span></td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td><span class="nonterminal">asset</span></td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td><span class="nonterminal">action</span></td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td><span class="nonterminal">transition</span></td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td><span class="nonterminal">dextension</span></td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td><span class="nonterminal">namespace</span></td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td><span class="nonterminal">contract</span></td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td><span class="nonterminal">function_decl</span></td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td><span class="nonterminal">verification_decl</span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">archetype</span></th>
        <td>ARCHETYPE
        <span class="option"><span class="nonterminal">extensions</span></span>
        <span class="nonterminal">ident</span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">vc_decl(X)</span></th>
        <td>X
        <span class="option"><span class="nonterminal">extensions</span></span>
        <span class="nonterminal">ident</span>
        <span class="nonterminal">type_t</span>
        <span class="option"><span class="nonterminal">value_options</span></span>
        <span class="option"><span class="nonterminal">default_value</span></span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">constant</span></th>
        <td><span class="nonterminal">vc_decl(CONSTANT)</span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">variable</span></th>
        <td><span class="nonterminal">vc_decl(VARIABLE)</span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">value_options</span></th>
        <td><span class="ne_list"><span class="nonterminal">value_option</span></span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">value_option</span></th>
        <td><span class="nonterminal">from_value</span></td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td><span class="nonterminal">to_value</span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">default_value</span></th>
        <td>EQUAL
        <span class="nonterminal">expr</span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">from_value</span></th>
        <td>FROM
        <span class="nonterminal">qualid</span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">to_value</span></th>
        <td>TO
        <span class="nonterminal">qualid</span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">dextension</span></th>
        <td>PERCENT
        <span class="nonterminal">ident</span>
        <span class="option">nonempty_list(<span class="nonterminal">simple_expr</span>)</span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">extensions</span></th>
        <td><span class="ne_list"><span class="nonterminal">extension</span></span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">extension</span></th>
        <td><span class="nonterminal">loc(<span class="nonterminal">extension_r</span>)</span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">extension_r</span></th>
        <td>LBRACKETPERCENT
        <span class="nonterminal">ident</span>
        <span class="option"><span class="ne_list"><span class="nonterminal">simple_expr</span></span></span>
        RBRACKET</td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">namespace</span></th>
        <td>NAMESPACE
        <span class="nonterminal">ident</span>
        <span class="nonterminal">braced(<span class="nonterminal">declarations</span>)</span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">contract</span></th>
        <td>CONTRACT
        <span class="option"><span class="nonterminal">extensions</span></span>
        <span class="nonterminal">ident</span>
        EQUAL
        <span class="nonterminal">braced(<span class="nonterminal">signatures</span>)</span>
        <span class="option"><span class="nonterminal">default_value</span></span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">signatures</span></th>
        <td><span class="ne_list"><span class="nonterminal">signature</span></span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">signature</span></th>
        <td>ACTION
        <span class="nonterminal">ident</span></td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td>ACTION
        <span class="nonterminal">ident</span>
        COLON
        <span class="nonterminal">types</span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">fun_effect</span></th>
        <td>EFFECT
        <span class="nonterminal">braced(<span class="nonterminal">expr</span>)</span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">fun_body</span></th>
        <td><span class="nonterminal">expr</span></td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td><span class="nonterminal">verification_fun</span>
        <span class="nonterminal">fun_effect</span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">function_gen</span></th>
        <td>FUNCTION
        <span class="nonterminal">ident</span>
        <span class="nonterminal">function_args</span>
        <span class="option"><span class="nonterminal">function_return</span></span>
        EQUAL
        LBRACE
        <span class="nonterminal">fun_body</span>
        RBRACE</td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">function_item</span></th>
        <td><span class="nonterminal">loc(<span class="nonterminal">function_gen</span>)</span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">function_decl</span></th>
        <td><span class="nonterminal">function_gen</span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">verif_predicate</span></th>
        <td>PREDICATE
        <span class="nonterminal">ident</span>
        <span class="nonterminal">function_args</span>
        EQUAL
        <span class="nonterminal">braced(<span class="nonterminal">expr</span>)</span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">verif_definition</span></th>
        <td>DEFINITION
        <span class="nonterminal">ident</span>
        EQUAL
        LBRACE
        <span class="nonterminal">ident</span>
        COLON
        <span class="nonterminal">type_t</span>
        PIPE
        <span class="nonterminal">expr</span>
        RBRACE</td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">verif_axiom</span></th>
        <td>AXIOM
        <span class="nonterminal">ident</span>
        EQUAL
        <span class="nonterminal">braced(<span class="nonterminal">expr</span>)</span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">verif_theorem</span></th>
        <td>THEOREM
        <span class="nonterminal">ident</span>
        EQUAL
        <span class="nonterminal">braced(<span class="nonterminal">expr</span>)</span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">verif_variable</span></th>
        <td>VARIABLE
        <span class="nonterminal">ident</span>
        <span class="nonterminal">type_t</span>
        <span class="option"><span class="nonterminal">default_value</span></span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">verif_invariant</span></th>
        <td>INVARIANT
        <span class="nonterminal">ident</span>
        EQUAL
        <span class="nonterminal">braced(<span class="nonterminal">expr</span>)</span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">verif_effect</span></th>
        <td>EFFECT
        <span class="nonterminal">braced(<span class="nonterminal">expr</span>)</span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">verif_specification</span></th>
        <td>SPECIFICATION
        <span class="nonterminal">braced(<span class="nonterminal">expr</span>)</span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">verif_item</span></th>
        <td><span class="nonterminal">verif_predicate</span></td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td><span class="nonterminal">verif_definition</span></td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td><span class="nonterminal">verif_axiom</span></td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td><span class="nonterminal">verif_theorem</span></td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td><span class="nonterminal">verif_variable</span></td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td><span class="nonterminal">verif_invariant</span></td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td><span class="nonterminal">verif_effect</span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">verification(spec)</span></th>
        <td><span class="nonterminal">loc(spec)</span></td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td>VERIFICATION
        <span class="option"><span class="nonterminal">extensions</span></span>
        LBRACE
        <span class="list"><span class="nonterminal">loc(<span class="nonterminal">verif_item</span>)</span></span>
        <span class="nonterminal">loc(<span class="nonterminal">verif_specification</span>)</span>
        RBRACE</td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">verification_fun</span></th>
        <td><span class="nonterminal">loc(<span class="nonterminal">verification(<span class="nonterminal">verif_specification</span>)</span>)</span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">verification_decl</span></th>
        <td><span class="nonterminal">loc(<span class="nonterminal">verification(<span class="nonterminal">verif_specification</span>)</span>)</span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">enum</span></th>
        <td>ENUM
        <span class="option"><span class="nonterminal">extensions</span></span>
        <span class="nonterminal">ident</span>
        EQUAL
        <span class="nonterminal">pipe_idents</span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">states</span></th>
        <td>STATES
        <span class="option"><span class="nonterminal">extensions</span></span>
        <span class="option"><span class="nonterminal">ident</span></span>
        <span class="option"><span class="nonterminal">states_values</span></span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">states_values</span></th>
        <td>EQUAL
        <span class="nonterminal">pipe_ident_options</span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">types</span></th>
        <td><span class="nonterminal">type_t</span><sup>+</sup><sub>COMMA</sub></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">type_t</span></th>
        <td><span class="nonterminal">loc(<span class="nonterminal">type_r</span>)</span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">type_r</span></th>
        <td><span class="nonterminal">type_s</span>
        <span class="nonterminal">type_tuples</span></td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td><span class="nonterminal">type_s_unloc</span></td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td><span class="nonterminal">type_t</span>
        IMPLY
        <span class="nonterminal">type_t</span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">type_s</span></th>
        <td><span class="nonterminal">loc(<span class="nonterminal">type_s_unloc</span>)</span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">type_s_unloc</span></th>
        <td><span class="nonterminal">ident</span></td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td><span class="nonterminal">type_s</span>
        <span class="nonterminal">container</span></td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td><span class="nonterminal">ident</span>
        <span class="nonterminal">type_s</span></td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td><span class="nonterminal">paren(<span class="nonterminal">type_r</span>)</span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">type_tuples</span></th>
        <td><span class="ne_list"><span class="nonterminal">type_tuple</span></span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">type_tuple</span></th>
        <td>MULT
        <span class="nonterminal">type_s</span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">container</span></th>
        <td>COLLECTION</td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td>QUEUE</td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td>STACK</td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td>SET</td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td>PARTITION</td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">pipe_idents</span></th>
        <td><span class="option">PIPE</span>
        <span class="nonterminal">ident</span><sup>+</sup><sub>PIPE</sub></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">pipe_ident_options</span></th>
        <td><span class="option">PIPE</span>
        <span class="nonterminal">pipe_ident_option</span><sup>+</sup><sub>PIPE</sub></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">pipe_ident_option</span></th>
        <td><span class="nonterminal">ident</span>
        <span class="option"><span class="nonterminal">state_options</span></span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">state_options</span></th>
        <td><span class="ne_list"><span class="nonterminal">state_option</span></span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">state_option</span></th>
        <td>INITIAL</td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td>WITH
        <span class="nonterminal">braced(<span class="nonterminal">expr</span>)</span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">asset</span></th>
        <td>ASSET
        <span class="option"><span class="nonterminal">extensions</span></span>
        <span class="option"><span class="nonterminal">bracket(<span class="nonterminal">asset_operation</span>)</span></span>
        <span class="nonterminal">ident</span>
        <span class="option"><span class="nonterminal">asset_options</span></span>
        <span class="option"><span class="nonterminal">asset_fields</span></span>
        <span class="nonterminal">asset_post_options</span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">asset_post_option</span></th>
        <td>WITH
        STATES
        <span class="nonterminal">ident</span></td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td>WITH
        <span class="nonterminal">braced(<span class="nonterminal">expr</span>)</span></td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td>INITIALIZED
        BY
        <span class="nonterminal">simple_expr</span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">asset_post_options</span></th>
        <td><span class="list"><span class="nonterminal">asset_post_option</span></span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">asset_fields</span></th>
        <td>EQUAL
        <span class="nonterminal">braced(<span class="nonterminal">fields</span>)</span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">asset_options</span></th>
        <td><span class="ne_list"><span class="nonterminal">asset_option</span></span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">asset_option</span></th>
        <td>AS
        <span class="nonterminal">ident</span></td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td>IDENTIFIED
        BY
        <span class="nonterminal">ident</span></td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td>SORTED
        BY
        <span class="nonterminal">ident</span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">fields</span></th>
        <td><span class="nonterminal">snl(SEMI_COLON,
        <span class="nonterminal">field</span>)</span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">field_r</span></th>
        <td><span class="nonterminal">ident</span>
        <span class="option"><span class="nonterminal">extensions</span></span>
        COLON
        <span class="nonterminal">type_t</span>
        <span class="option">REF</span>
        <span class="option"><span class="nonterminal">default_value</span></span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">field</span></th>
        <td><span class="nonterminal">loc(<span class="nonterminal">field_r</span>)</span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">ident</span></th>
        <td><span class="nonterminal">loc(IDENT)</span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">action</span></th>
        <td>ACTION
        <span class="option"><span class="nonterminal">extensions</span></span>
        <span class="nonterminal">ident</span>
        <span class="nonterminal">function_args</span>
        <span class="nonterminal">transitems_eq</span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">transition_to_item</span></th>
        <td>TO
        <span class="nonterminal">ident</span>
        <span class="option"><span class="nonterminal">require_value</span></span>
        <span class="option"><span class="nonterminal">with_effect</span></span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">transitions</span></th>
        <td><span class="ne_list"><span class="nonterminal">transition_to_item</span></span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">on_value</span></th>
        <td>ON
        <span class="nonterminal">ident</span>
        COLON
        <span class="nonterminal">ident</span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">transition</span></th>
        <td>TRANSITION
        <span class="option"><span class="nonterminal">extensions</span></span>
        <span class="nonterminal">ident</span>
        <span class="nonterminal">function_args</span>
        <span class="option"><span class="nonterminal">on_value</span></span>
        FROM
        <span class="nonterminal">expr</span>
        EQUAL
        LBRACE
        <span class="nonterminal">action_properties</span>
        <span class="nonterminal">transitions</span>
        RBRACE</td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">transitems_eq</span></th>
        <td><span class="option">EQUAL
        LBRACE
        <span class="nonterminal">action_properties</span>
        <span class="option"><span class="nonterminal">effect</span></span>
        RBRACE</span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">action_properties</span></th>
        <td><span class="option"><span class="nonterminal">calledby</span></span>
        <span class="option">ACCEPT_TRANSFER</span>
        <span class="option"><span class="nonterminal">require</span></span>
        <span class="option"><span class="nonterminal">verification_fun</span></span>
        <span class="list"><span class="nonterminal">function_item</span></span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">calledby</span></th>
        <td>CALLED
        BY
        <span class="option"><span class="nonterminal">extensions</span></span>
        <span class="nonterminal">expr</span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">require</span></th>
        <td>REQUIRE
        <span class="option"><span class="nonterminal">extensions</span></span>
        <span class="nonterminal">braced(<span class="nonterminal">expr</span>)</span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">require_value</span></th>
        <td>WHEN
        <span class="option"><span class="nonterminal">extensions</span></span>
        <span class="nonterminal">braced(<span class="nonterminal">expr</span>)</span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">with_effect</span></th>
        <td>WITH
        EFFECT
        <span class="option"><span class="nonterminal">extensions</span></span>
        <span class="nonterminal">braced(<span class="nonterminal">expr</span>)</span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">effect</span></th>
        <td>EFFECT
        <span class="option"><span class="nonterminal">extensions</span></span>
        <span class="nonterminal">braced(<span class="nonterminal">expr</span>)</span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">function_return</span></th>
        <td>COLON
        <span class="nonterminal">type_t</span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">function_args</span></th>
        <td><span class="option"><span class="ne_list"><span class="nonterminal">function_arg</span></span></span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">function_arg</span></th>
        <td><span class="nonterminal">ident</span>
        <span class="option"><span class="nonterminal">extensions</span></span></td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td>LPAREN
        <span class="nonterminal">ident</span>
        <span class="option"><span class="nonterminal">extensions</span></span>
        COLON
        <span class="nonterminal">type_t</span>
        RPAREN</td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">assignment_operator_record</span></th>
        <td>EQUAL</td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td><span class="nonterminal">assignment_operator_extra</span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">assignment_operator_expr</span></th>
        <td>COLONEQUAL</td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td><span class="nonterminal">assignment_operator_extra</span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">assignment_operator_extra</span></th>
        <td>PLUSEQUAL</td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td>MINUSEQUAL</td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td>MULTEQUAL</td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td>DIVEQUAL</td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td>ANDEQUAL</td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td>OREQUAL</td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">qualid</span></th>
        <td><span class="nonterminal">ident</span><sup>+</sup><sub>DOT</sub></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">branchs</span></th>
        <td><span class="ne_list"><span class="nonterminal">branch</span></span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">branch</span></th>
        <td><span class="nonterminal">patterns</span>
        IMPLY
        <span class="nonterminal">expr</span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">patterns</span></th>
        <td><span class="ne_list"><span class="nonterminal">loc(<span class="nonterminal">pattern</span>)</span></span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">pattern</span></th>
        <td>PIPE
        UNDERSCORE</td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td>PIPE
        <span class="nonterminal">ident</span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">expr</span></th>
        <td><span class="nonterminal">loc(<span class="nonterminal">expr_r</span>)</span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">ident_typ_q_item</span></th>
        <td>LPAREN
        <span class="ne_list"><span class="nonterminal">ident</span></span>
        COLON
        <span class="nonterminal">type_t</span>
        RPAREN</td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">ident_typ_q</span></th>
        <td><span class="ne_list"><span class="nonterminal">ident_typ_q_item</span></span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">expr_r</span></th>
        <td><span class="nonterminal">quantifier</span>
        <span class="nonterminal">ident_typ1</span>
        COMMA
        <span class="nonterminal">expr</span></td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td><span class="nonterminal">quantifier</span>
        <span class="nonterminal">ident_typ_q</span>
        COMMA
        <span class="nonterminal">expr</span></td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td>LET
        <span class="nonterminal">ident_typ1</span>
        EQUAL
        <span class="nonterminal">expr</span>
        IN
        <span class="nonterminal">expr</span>
        OTHERWISE
        <span class="nonterminal">expr</span></td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td>LET
        <span class="nonterminal">ident_typ1</span>
        EQUAL
        <span class="nonterminal">expr</span>
        IN
        <span class="nonterminal">expr</span></td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td><span class="nonterminal">expr</span>
        SEMI_COLON
        <span class="nonterminal">expr</span></td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td><span class="nonterminal">ident</span>
        COLON
        <span class="nonterminal">expr</span></td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td>FUN
        <span class="nonterminal">ident_typs</span>
        EQUALGREATER
        <span class="nonterminal">expr</span></td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td>ASSERT
        <span class="nonterminal">paren(<span class="nonterminal">expr</span>)</span></td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td>BREAK</td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td>FOR
        LPAREN
        <span class="nonterminal">ident</span>
        IN
        <span class="nonterminal">expr</span>
        RPAREN
        <span class="nonterminal">expr</span></td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td>IF
        <span class="nonterminal">expr</span>
        THEN
        <span class="nonterminal">expr</span></td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td>IF
        <span class="nonterminal">expr</span>
        THEN
        <span class="nonterminal">expr</span>
        ELSE
        <span class="nonterminal">expr</span></td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td>MATCH
        <span class="nonterminal">expr</span>
        WITH
        <span class="nonterminal">branchs</span>
        END</td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td><span class="nonterminal">snl2(COMMA,
        <span class="nonterminal">simple_expr</span>)</span></td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td><span class="nonterminal">expr</span>
        <span class="nonterminal">assignment_operator_expr</span>
        <span class="nonterminal">expr</span></td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td>TRANSFER
        <span class="option">BACK</span>
        <span class="nonterminal">simple_expr</span>
        <span class="option"><span class="nonterminal">to_value</span></span></td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td>REQUIRE
        <span class="nonterminal">simple_expr</span></td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td>FAILIF
        <span class="nonterminal">simple_expr</span></td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td><span class="nonterminal">order_operations</span></td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td><span class="nonterminal">expr</span>
        <span class="nonterminal">loc(<span class="nonterminal">bin_operator</span>)</span>
        <span class="nonterminal">expr</span></td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td><span class="nonterminal">loc(<span class="nonterminal">un_operator</span>)</span>
        <span class="nonterminal">expr</span></td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td><span class="nonterminal">ident</span>
        <span class="nonterminal">app_args</span></td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td><span class="nonterminal">simple_expr</span>
        DOT
        <span class="nonterminal">ident</span>
        <span class="nonterminal">app_args</span></td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td><span class="nonterminal">simple_expr_r</span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">order_operation</span></th>
        <td><span class="nonterminal">expr</span>
        <span class="nonterminal">loc(<span class="nonterminal">ord_operator</span>)</span>
        <span class="nonterminal">expr</span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">order_operations</span></th>
        <td><span class="nonterminal">order_operation</span></td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td><span class="nonterminal">loc(<span class="nonterminal">order_operations</span>)</span>
        <span class="nonterminal">loc(<span class="nonterminal">ord_operator</span>)</span>
        <span class="nonterminal">expr</span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">app_args</span></th>
        <td>LPAREN
        RPAREN</td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td><span class="ne_list"><span class="nonterminal">simple_expr</span></span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">simple_expr</span></th>
        <td><span class="nonterminal">loc(<span class="nonterminal">simple_expr_r</span>)</span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">simple_expr_r</span></th>
        <td><span class="nonterminal">simple_expr</span>
        DOT
        <span class="nonterminal">ident</span></td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td>LBRACKET
        RBRACKET</td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td>LBRACKET
        <span class="nonterminal">expr</span>
        RBRACKET</td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td>LBRACE
        <span class="nonterminal">record_item</span><sup>+</sup><sub>SEMI_COLON</sub>
        RBRACE</td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td><span class="nonterminal">literal</span></td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td><span class="nonterminal">ident</span>
        COLONCOLON
        <span class="nonterminal">ident</span></td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td><span class="nonterminal">ident</span></td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td>LPAREN
        <span class="nonterminal">ident</span>
        COLONCOLON
        <span class="nonterminal">ident</span>
        AT
        <span class="nonterminal">ident</span>
        RPAREN</td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td>LPAREN
        <span class="nonterminal">ident</span>
        AT
        <span class="nonterminal">ident</span>
        RPAREN</td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td><span class="nonterminal">paren(<span class="nonterminal">expr_r</span>)</span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">ident_typs</span></th>
        <td><span class="ne_list"><span class="nonterminal">ident</span></span>
        COLON
        <span class="nonterminal">type_t</span></td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td><span class="ne_list"><span class="nonterminal">ident_typ</span></span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">ident_typ</span></th>
        <td><span class="nonterminal">ident</span></td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td>LPAREN
        <span class="ne_list"><span class="nonterminal">ident</span></span>
        COLON
        <span class="nonterminal">type_t</span>
        RPAREN</td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">ident_typ1</span></th>
        <td><span class="nonterminal">ident</span>
        <span class="option">COLON
        <span class="nonterminal">type_t</span></span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">literal</span></th>
        <td>NUMBER</td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td>RATIONAL</td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td>TZ</td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td>STRING</td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td>ADDRESS</td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td><span class="nonterminal">bool_value</span></td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td>DURATION</td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td>DATE</td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">bool_value</span></th>
        <td>TRUE</td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td>FALSE</td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">record_item</span></th>
        <td><span class="nonterminal">simple_expr</span></td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td><span class="nonterminal">ident</span>
        <span class="nonterminal">assignment_operator_record</span>
        <span class="nonterminal">simple_expr</span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">quantifier</span></th>
        <td>FORALL</td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td>EXISTS</td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">spec_operator</span></th>
        <td>OP_SPEC1</td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td>OP_SPEC2</td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td>OP_SPEC3</td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td>OP_SPEC4</td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">logical_operator</span></th>
        <td>AND</td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td>OR</td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td>IMPLY</td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td>EQUIV</td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">comparison_operator</span></th>
        <td>EQUAL</td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td>NEQUAL</td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">ordering_operator</span></th>
        <td>LESS</td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td>LESSEQUAL</td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td>GREATER</td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td>GREATEREQUAL</td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">arithmetic_operator</span></th>
        <td>PLUS</td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td>MINUS</td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td>MULT</td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td>DIV</td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td>PERCENT</td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">unary_operator</span></th>
        <td>PLUS</td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td>MINUS</td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td>NOT</td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">bin_operator</span></th>
        <td><span class="nonterminal">spec_operator</span></td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td><span class="nonterminal">logical_operator</span></td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td><span class="nonterminal">comparison_operator</span></td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td><span class="nonterminal">arithmetic_operator</span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">un_operator</span></th>
        <td><span class="nonterminal">unary_operator</span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">ord_operator</span></th>
        <td><span class="nonterminal">ordering_operator</span></td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">asset_operation_enum</span></th>
        <td>AT_ADD</td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td>AT_REMOVE</td>
      </tr>
      <tr>
        <th class="bar"></th>
        <td>AT_UPDATE</td>
      </tr>
      
      <tr class="rule">
        <th><span class="nonterminal">asset_operation</span></th>
        <td><span class="ne_list"><span class="nonterminal">asset_operation_enum</span></span>
        <span class="option"><span class="ne_list"><span class="nonterminal">simple_expr</span></span></span></td>
      </tr>
      
      
    </table>


```ocaml
 <loc(X)> ::= X

<snl(separator, X)> ::= X
                      | X separator
                      | X separator <snl(separator, X)>

<snl2(separator, X)> ::= X separator X
                       | X separator <snl2(separator, X)>

<paren(X)> ::= LPAREN X RPAREN

<braced(X)> ::= LBRACE X RBRACE

<bracket(X)> ::= LBRACKET X RBRACKET

<start_expr> ::= <expr> EOF
               | <loc(error)>

<main> ::= <loc(<archetype_r>)>
         | <loc(error)>

<archetype_r> ::= <implementation_archetype> EOF
                | <archetype_extension> EOF

<archetype_extension> ::= ARCHETYPE EXTENSION <ident> <paren(<declarations>)>
                          EQUAL <braced(<declarations>)>

<implementation_archetype> ::= <declarations>

<declarations> ::= <declaration>+

<declaration> ::= <loc(<declaration_r>)>

<declaration_r> ::= <archetype>
                  | <constant>
                  | <variable>
                  | <enum>
                  | <states>
                  | <asset>
                  | <action>
                  | <transition>
                  | <dextension>
                  | <namespace>
                  | <contract>
                  | <function_decl>
                  | <verification_decl>

<archetype> ::= ARCHETYPE [<extensions>] <ident>

<vc_decl(X)> ::= X [<extensions>] <ident> <type_t> [<value_options>]
                 [<default_value>]

<constant> ::= <vc_decl(CONSTANT)>

<variable> ::= <vc_decl(VARIABLE)>

<value_options> ::= <value_option>+

<value_option> ::= <from_value>
                 | <to_value>

<default_value> ::= EQUAL <expr>

<from_value> ::= FROM <qualid>

<to_value> ::= TO <qualid>

<dextension> ::= PERCENT <ident> [nonempty_list(<simple_expr>)]

<extensions> ::= <extension>+

<extension> ::= <loc(<extension_r>)>

<extension_r> ::= LBRACKETPERCENT <ident> [<simple_expr>+] RBRACKET

<namespace> ::= NAMESPACE <ident> <braced(<declarations>)>

<contract> ::= CONTRACT [<extensions>] <ident> EQUAL <braced(<signatures>)>
               [<default_value>]

<signatures> ::= <signature>+

<signature> ::= ACTION <ident>
              | ACTION <ident> COLON <types>

<fun_effect> ::= EFFECT <braced(<expr>)>

<fun_body> ::= <expr>
             | <verification_fun> <fun_effect>

<function_gen> ::= FUNCTION <ident> <function_args> [<function_return>] EQUAL
                   LBRACE <fun_body> RBRACE

<function_item> ::= <loc(<function_gen>)>

<function_decl> ::= <function_gen>

<verif_predicate> ::= PREDICATE <ident> <function_args> EQUAL
                      <braced(<expr>)>

<verif_definition> ::= DEFINITION <ident> EQUAL LBRACE <ident> COLON <type_t>
                       PIPE <expr> RBRACE

<verif_axiom> ::= AXIOM <ident> EQUAL <braced(<expr>)>

<verif_theorem> ::= THEOREM <ident> EQUAL <braced(<expr>)>

<verif_variable> ::= VARIABLE <ident> <type_t> [<default_value>]

<verif_invariant> ::= INVARIANT <ident> EQUAL <braced(<expr>)>

<verif_effect> ::= EFFECT <braced(<expr>)>

<verif_specification> ::= SPECIFICATION <braced(<expr>)>

<verif_item> ::= <verif_predicate>
               | <verif_definition>
               | <verif_axiom>
               | <verif_theorem>
               | <verif_variable>
               | <verif_invariant>
               | <verif_effect>

<verification(spec)> ::= <loc(spec)>
                       | VERIFICATION [<extensions>] LBRACE
                         <loc(<verif_item>)>* <loc(<verif_specification>)>
                         RBRACE

<verification_fun> ::= <loc(<verification(<verif_specification>)>)>

<verification_decl> ::= <loc(<verification(<verif_specification>)>)>

<enum> ::= ENUM [<extensions>] <ident> EQUAL <pipe_idents>

<states> ::= STATES [<extensions>] [<ident>] [<states_values>]

<states_values> ::= EQUAL <pipe_ident_options>

<types> ::= <type_t> (COMMA <type_t>)*

<type_t> ::= <loc(<type_r>)>

<type_r> ::= <type_s> <type_tuples>
           | <type_s_unloc>
           | <type_t> IMPLY <type_t>

<type_s> ::= <loc(<type_s_unloc>)>

<type_s_unloc> ::= <ident>
                 | <type_s> <container>
                 | <ident> <type_s>
                 | <paren(<type_r>)>

<type_tuples> ::= <type_tuple>+

<type_tuple> ::= MULT <type_s>

<container> ::= COLLECTION
              | QUEUE
              | STACK
              | SET
              | PARTITION

<pipe_idents> ::= [PIPE] <ident> (PIPE <ident>)*

<pipe_ident_options> ::= [PIPE] <pipe_ident_option> (PIPE
                         <pipe_ident_option>)*

<pipe_ident_option> ::= <ident> [<state_options>]

<state_options> ::= <state_option>+

<state_option> ::= INITIAL
                 | WITH <braced(<expr>)>

<asset> ::= ASSET [<extensions>] [<bracket(<asset_operation>)>] <ident>
            [<asset_options>] [<asset_fields>] <asset_post_options>

<asset_post_option> ::= WITH STATES <ident>
                      | WITH <braced(<expr>)>
                      | INITIALIZED BY <simple_expr>

<asset_post_options> ::= <asset_post_option>*

<asset_fields> ::= EQUAL <braced(<fields>)>

<asset_options> ::= <asset_option>+

<asset_option> ::= AS <ident>
                 | IDENTIFIED BY <ident>
                 | SORTED BY <ident>

<fields> ::= <snl(SEMI_COLON, <field>)>

<field_r> ::= <ident> [<extensions>] COLON <type_t> [REF] [<default_value>]

<field> ::= <loc(<field_r>)>

<ident> ::= <loc(IDENT)>

<action> ::= ACTION [<extensions>] <ident> <function_args> <transitems_eq>

<transition_to_item> ::= TO <ident> [<require_value>] [<with_effect>]

<transitions> ::= <transition_to_item>+

<on_value> ::= ON <ident> COLON <ident>

<transition> ::= TRANSITION [<extensions>] <ident> <function_args>
                 [<on_value>] FROM <expr> EQUAL LBRACE <action_properties>
                 <transitions> RBRACE

<transitems_eq> ::= [EQUAL LBRACE <action_properties> [<effect>] RBRACE]

<action_properties> ::= [<calledby>] [ACCEPT_TRANSFER] [<require>]
                        [<verification_fun>] <function_item>*

<calledby> ::= CALLED BY [<extensions>] <expr>

<require> ::= REQUIRE [<extensions>] <braced(<expr>)>

<require_value> ::= WHEN [<extensions>] <braced(<expr>)>

<with_effect> ::= WITH EFFECT [<extensions>] <braced(<expr>)>

<effect> ::= EFFECT [<extensions>] <braced(<expr>)>

<function_return> ::= COLON <type_t>

<function_args> ::= [<function_arg>+]

<function_arg> ::= <ident> [<extensions>]
                 | LPAREN <ident> [<extensions>] COLON <type_t> RPAREN

<assignment_operator_record> ::= EQUAL
                               | <assignment_operator_extra>

<assignment_operator_expr> ::= COLONEQUAL
                             | <assignment_operator_extra>

<assignment_operator_extra> ::= PLUSEQUAL
                              | MINUSEQUAL
                              | MULTEQUAL
                              | DIVEQUAL
                              | ANDEQUAL
                              | OREQUAL

<qualid> ::= <ident> (DOT <ident>)*

<branchs> ::= <branch>+

<branch> ::= <patterns> IMPLY <expr>

<patterns> ::= <loc(<pattern>)>+

<pattern> ::= PIPE UNDERSCORE
            | PIPE <ident>

<expr> ::= <loc(<expr_r>)>

<ident_typ_q_item> ::= LPAREN <ident>+ COLON <type_t> RPAREN

<ident_typ_q> ::= <ident_typ_q_item>+

<expr_r> ::= <quantifier> <ident_typ1> COMMA <expr>
           | <quantifier> <ident_typ_q> COMMA <expr>
           | LET <ident_typ1> EQUAL <expr> IN <expr> OTHERWISE <expr>
           | LET <ident_typ1> EQUAL <expr> IN <expr>
           | <expr> SEMI_COLON <expr>
           | <ident> COLON <expr>
           | FUN <ident_typs> EQUALGREATER <expr>
           | ASSERT <paren(<expr>)>
           | BREAK
           | FOR LPAREN <ident> IN <expr> RPAREN <expr>
           | IF <expr> THEN <expr>
           | IF <expr> THEN <expr> ELSE <expr>
           | MATCH <expr> WITH <branchs> END
           | <snl2(COMMA, <simple_expr>)>
           | <expr> <assignment_operator_expr> <expr>
           | TRANSFER [BACK] <simple_expr> [<to_value>]
           | REQUIRE <simple_expr>
           | FAILIF <simple_expr>
           | <order_operations>
           | <expr> <loc(<bin_operator>)> <expr>
           | <loc(<un_operator>)> <expr>
           | <ident> <app_args>
           | <simple_expr> DOT <ident> <app_args>
           | <simple_expr_r>

<order_operation> ::= <expr> <loc(<ord_operator>)> <expr>

<order_operations> ::= <order_operation>
                     | <loc(<order_operations>)> <loc(<ord_operator>)> <expr>

<app_args> ::= LPAREN RPAREN
             | <simple_expr>+

<simple_expr> ::= <loc(<simple_expr_r>)>

<simple_expr_r> ::= <simple_expr> DOT <ident>
                  | LBRACKET RBRACKET
                  | LBRACKET <expr> RBRACKET
                  | LBRACE <record_item> (SEMI_COLON <record_item>)* RBRACE
                  | <literal>
                  | <ident> COLONCOLON <ident>
                  | <ident>
                  | LPAREN <ident> COLONCOLON <ident> AT <ident> RPAREN
                  | LPAREN <ident> AT <ident> RPAREN
                  | <paren(<expr_r>)>

<ident_typs> ::= <ident>+ COLON <type_t>
               | <ident_typ>+

<ident_typ> ::= <ident>
              | LPAREN <ident>+ COLON <type_t> RPAREN

<ident_typ1> ::= <ident> [COLON <type_t>]

<literal> ::= NUMBER
            | RATIONAL
            | TZ
            | STRING
            | ADDRESS
            | <bool_value>
            | DURATION
            | DATE

<bool_value> ::= TRUE
               | FALSE

<record_item> ::= <simple_expr>
                | <ident> <assignment_operator_record> <simple_expr>

<quantifier> ::= FORALL
               | EXISTS

<spec_operator> ::= OP_SPEC1
                  | OP_SPEC2
                  | OP_SPEC3
                  | OP_SPEC4

<logical_operator> ::= AND
                     | OR
                     | IMPLY
                     | EQUIV

<comparison_operator> ::= EQUAL
                        | NEQUAL

<ordering_operator> ::= LESS
                      | LESSEQUAL
                      | GREATER
                      | GREATEREQUAL

<arithmetic_operator> ::= PLUS
                        | MINUS
                        | MULT
                        | DIV
                        | PERCENT

<unary_operator> ::= PLUS
                   | MINUS
                   | NOT

<bin_operator> ::= <spec_operator>
                 | <logical_operator>
                 | <comparison_operator>
                 | <arithmetic_operator>

<un_operator> ::= <unary_operator>

<ord_operator> ::= <ordering_operator>

<asset_operation_enum> ::= AT_ADD
                         | AT_REMOVE
                         | AT_UPDATE

<asset_operation> ::= <asset_operation_enum>+ [<simple_expr>+]

```

