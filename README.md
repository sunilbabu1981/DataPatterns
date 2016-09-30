###DataPatterns

DataPatterns is an ECL bundle that provides some basic data profiling and research tools to an ECL programmer.

###License and Version
This software is licensed under the Apache v2 license.  A link to the license, as well as the current version of this software, can be found in the [Bundle.ecl](https://github.com/hpcc-systems/DataPatterns/blob/master/Bundle.ecl) file.

###Installation

This code is installed as an ECL Bundle.  Complete instructions for managing ECL Bundles can be found in [The ECL IDE and HPCC Client Tools](http://cdn.hpccsystems.com/releases/CE-Candidate-6.0.6/docs/TheECLIDEandHPCCClientTools-6.0.6-1.pdf) documentation.

Use the ecl command line tool to install this bundle:

	ecl bundle install https://github.com/hpcc-systems/DataPatterns.git

You may have to either navigate to the client tools bin directory before executing the command, or use the full path to the ecl tool.

After installation, all of the code here becomes available after you import it:

	IMPORT DataPatterns;

###Profile

Documentation as pulled from the beginning of Profile.ecl:

	Profile() is a function macro for profiling all or part of a dataset.
	The output is a dataset containing the following information for each
	profiled attribute:

		 attribute               The name of the attribute
		 given_attribute_type    The ECL type of the attribute as it was defined
								 in the input dataset
		 rec_count               The number of records in the dataset
		 fill_rate               The percentage of records containing non-nil
								 values; a 'nil value' is an empty string or
								 a numeric zero; note that BOOLEAN attributes
								 are always counted as filled, regardless of
								 their value
		 cardinality             The number of unique, non-nil values within
								 the attribute
		 best_attribute_type     And ECL data type that both allows all values
								 in the input dataset and consumes the least
								 amount of memory
		 modes                   The most common values in the attribute, after
								 coercing all values to STRING, along with the
								 number of records in which the values were
								 found; if no value is repeated more than once
								 then no mode will be shown; up to five (5)
								 modes will be shown; note that string values
								 longer than the maxPatternLen argument will
								 be truncated
		 min_length              The shortest length of a value when expressed
								 as a string; null values are ignored
		 max_length              The longest length of a value when expressed
								 as a string
		 ave_length              The average length of a value when expressed
								 as a string
		 popular_patterns        The most common patterns of values; see below
		 rare_patterns           The least common patterns of values; see below
		 is_numeric              Boolean indicating if the original attribute
								 was numeric and therefore whether or not
								 the numeric_xxxx output fields will be
								 populated with actual values; if this value
								 is FALSE then all numeric_xxxx output values
								 should be ignored
		 numeric_min             The smallest non-nil value found within the
								 attribute as a DECIMAL; the attribute must be
								 a numeric ECL datatype; non-numeric attributes
								 will return zero
		 numeric_max             The largest non-nil value found within the
								 attribute as a DECIMAL; the attribute must be
								 a numeric ECL datatype; non-numeric attributes
								 will return zero
		 numeric_mean            The mean (average) non-nil value found within
								 the attribute as a DECIMAL; the attribute must
								 be a numeric ECL datatype; non-numeric
								 attributes will return zero
		 numeric_std_dev         The standard deviation of the non-nil values
								 in the attribute as a DECIMAL; the attribute
								 must be a numeric ECL datatype; non-numeric
								 attributes will return zero
		 numeric_first_quartile  The value separating the first (bottom) and
								 second quarters of non-nil values within
								 the attribute as a DECIMAL; the attribute must
								 be a numeric ECL datatype; non-numeric
								 attributes will return zero
		 numeric_median          The median non-nil value within the attribute
								 as a DECIMAL; the attribute must be a numeric
								 ECL datatype; non-numeric attributes will return
								 zero
		 numeric_third_quartile  The value separating the third and fourth
								 (top) quarters of non-nil values within
								 the attribute as a DECIMAL; the attribute must
								 be a numeric ECL datatype; non-numeric
								 attributes will return zero
		 numeric_correlations    A child dataset containing correlation values
								 comparing the current numeric attribute with all
								 other numeric attributes, listed in descending
								 correlation value order; the attribute must be
								 a numeric ECL datatype; non-numeric attributes
								 will return an empty child dataset; note that
								 this can be a time-consuming operation,
								 depending on the number of numeric attributes
								 in your dataset and the number of rows (if you
								 have N numeric attributes, then
								 N * (N - 1) / 2 calculations are performed,
								 each scanning all data rows)

	Most profile outputs can be disabled (though the record structure of the
	result does not change).  See the 'features' argument, below.

	Data patterns can give you an idea of what your data looks like when it is
	expressed as a (human-readable) string.  The function converts each
	character of the string into a fixed character palette to producing a "data
	pattern" and then counts the number of unique patterns for that attribute.
	The character palette used is:

		 A   Any uppercase letter
		 a   Any lowercase letter
		 9   Any numeric digit
		 B   A boolean value (true or false)

	All other characters are left as-is in the pattern.

	Only the top level attributes within a dataset are processed; embedded
	records and child recordsets are skipped.  SET data types (such as
	SET OF INTEGER) are also skipped.  If the dataset contains only
	embedded records and/or child recordsets, or if fieldListStr is specified
	but with attributes that don't actually exist in the top level (or are
	invalid) then an error will be produced during compilation time.

	This function works best when the incoming dataset contains attributes that
	have precise data types (e.g. UNSIGNED4 data types instead of numbers
	stored in a STRING data type).

	Function parameters:

	@param   inFile          The dataset to process; REQUIRED
	@param   fieldListStr    A string containing a comma-delimited list of
							 attribute names to process; use an empty string to
							 process all attributes in inFile; attributes named
							 here that are not found in the top level of inFile
							 will be ignored; OPTIONAL, defaults to an
							 empty string
	@param   maxPatterns     The maximum number of patterns (both popular and
							 rare) to return for each attribute; OPTIONAL,
							 defaults to 100
	@param   maxPatternLen   The maximum length of a pattern; longer patterns
							 are truncated in the output; this value is also
							 used to set the maximum length of the data to
							 consider when finding cardinality and mode values;
							 must be 33 or larger; OPTIONAL, defaults to 100
	@param   features        A comma-delimited string listing the profiling
							 elements to be included in the output; OPTIONAL,
							 defaults to a comma-delimited string containing all
							 of the available keywords:
								 KEYWORD         AFFECTED OUTPUT
								 fill_rate       fill_rate
								 best_ecl_types  best_attribute_type
								 cardinality     cardinality
								 modes           modes
								 lengths         min_length
												 max_length
												 ave_length
								 patterns        popular_patterns
												 rare_patterns
								 min_max         numeric_min
												 numeric_max
								 mean            numeric_mean
								 std_dev         numeric_std_dev
								 quartiles       numeric_first_quartile
												 numeric_median
												 numeric_third_quartile
								 correlations    numeric_correlations
							 To omit the output associated with a single keyword,
							 set this argument to a comma-delimited string
							 containing all other keywords; note that omitting a
							 profiling element does not change the resulting
							 record structure; the omitted element will simply
							 contain zeroes, empty strings, or empty child
							 datasets; this function will always output the
							 attribute name, given attribute type, and the record
							 count, no matter what features are chosen

Here is a very simple example of executing the full data profiling code:

	IMPORT DataPatterns;

	DataRec := { string source, unsigned bid, string name };

	ds := DATASET('~thor::my_sample_data', DataRec, FLAT);

	profileResults := DataPatterns.Profile(ds);

	OUTPUT(profileResults, ALL, NAMED('profileResults'));